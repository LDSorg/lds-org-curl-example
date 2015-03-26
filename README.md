READ THIS FIRST
===============

You should not need to access the LDS.org API directly with cURL.

Consider using using the **OAuth2 API at <https://lds.io>** instead.

This tutorial is provided for the sake of prototyping and exploring, not for production use.

OAuth2 API at <https://lds.io>
=================

Consider using the OAuth2 API. There are complete zero-config examples that can work from a single install script.

Exploring the LDS.org Mobile API with cURL
=================

**NOTE**: Some APIs have a trailing slash. Some don't. **It matters which is which**.

The Mobile API is listed here:

<https://tech.lds.org/mobile/ldstools/config.json>

And there is also some documentation available here (scroll to the bottom):

<http://tech.lds.org/wiki/LDS_Tools_Web_Services>

Many of the resources are only for leadership or are old resources that exist for backwards compatibility with old apps on platforms that are no longer supported (maybe Blackberry 10 / webOS) API

1. Login
--------

Assuming a valid login, this will set the appropriate cookies into `./my-credentials.txt` that you will need to make any other authenticated requests.

For the ease of copy and paste, first set your username and passphrase as variables in your shell:

```bash
LDS_USERNAME='joesmith'
LDS_PASSPHRASE='super secret'
```

Then proceed to login:

```bash
curl https://signin.lds.org/login.html \
  --request POST \
  --location \
  --cookie-jar ./my-session.txt \
  --data "username=${LDS_USERNAME}&password=${LDS_PASSPHRASE}"
```

The shorthand version of that command would be:

```bash
curl https://signin.lds.org/login.html \
  -X POST \
  -L \
  -c ./my-session.txt \
  -d "username=${USERNAME}&password=${PASSPHRASE}"
```

2. Your Profile
---------------

Since it takes several queries and a decent number of loops to get a full profile (such as your calling), we'll start with the bare minimum and work from there.

```bash
curl https://www.lds.org/mobiledirectory/services/v2/ldstools/current-user-detail \
  --cookie ./my-session.txt \
  --cookie-jar ./my-session.txt
```

Or, in shorthand:

```bash
curl https://www.lds.org/mobiledirectory/services/v2/ldstools/current-user-detail \
  -b ./my-session.txt \
  -c ./my-session.txt
```

That will give you your `homeUnitNbr` and `individualId` which you will need for the next request

```bash
LDS_INDIVIDUAL_ID='1234567890'
LDS_HOME_WARD_ID='123456'
```

3. Ward Directory - Contacts and Callings
------------

**emails** and **phone numbers**

```bash
curl "https://www.lds.org/mobiledirectory/services/v2/ldstools/member-detaillist-with-callings/${LDS_HOME_WARD_ID}" \
  --cookie ./my-session.txt \
  --cookie-jar ./my-session.txt
```

It can be helpful to pipe this through `python -m json.tool` and save it to a file for easy reference:

```bash
curl "https://www.lds.org/mobiledirectory/services/v2/ldstools/member-detaillist-with-callings/${LDS_HOME_WARD_ID}" \
  --cookie ./my-session.txt \
  --cookie-jar ./my-session.txt \
  | python -m json.tool > ./my-ward.json
```

**NOTE**: In a YSA ward the Bishopric (external home ward) will be included, but their ids will be malformed (no household id).

4. Stake Directory - Contacts and Callings
------------

Same as the above, except substituting `${LDS_HOME_STAKE_ID}` instead of `${LDS_HOME_WARD_ID}`.

```bash
curl "https://www.lds.org/mobiledirectory/services/v2/ldstools/member-detaillist-with-callings/${LDS_HOME_STAKE_ID}" \
  -b ./my-session.txt \
  -c ./my-session.txt \
  | python -m json.tool > ./my-ward.json
```

5. Familiy and Individual Photos
------------

### Family Photo

```bash
curl "https://lds.org/directory/services/ludrs/photo/url/${LDS_INDIVIDUAL_ID}/household" \
  --cookie ./my-session.txt \
  --cookie-jar ./my-session.txt
```

**NOTE**: The `houseOfHouseholdId` is *usually* the same as the `individualId`, but it's actually the church member of the household, which may be the wife or one of the children.

### Family Photo Directory

This is the web API, not the mobile API, but it'll give you all of the family photo urls in one shot, which is a plus.

```bash
curl "https://www.lds.org/directory/services/ludrs/mem/wardDirectory/photos/${LDS_HOME_WARD_ID}" \
  --cookie ./my-session.txt \
  --cookie-jar ./my-session.txt
```

### Individual Photo

```bash
curl "https://lds.org/directory/services/ludrs/photo/url/${LDS_INDIVIDUAL_ID}/individual" \
  --cookie ./my-session.txt \
  --cookie-jar ./my-session.txt
```

**NOTE**: I have no idea how you get the **Bishopric's Photos** in a **YSA ward**...

### Individual Photo Directory

Instead of making hundreds of requests, I've successfully tested passing in about 170 ids at a time using commas:

```bash
curl "https://lds.org/directory/services/ludrs/photo/url/${ID_1},${ID_2},${ID_3}/individual" \
  --cookie ./my-session.txt \
  --cookie-jar ./my-session.txt
```

Here's the script I used to make a large request from the `member-detaillist-with-callings` resources:

```javascript
var ward = require('./ward.json');
var ids = [];

ward.households.forEach(function (h) {
  if (h.headOfHouse && h.headOfHouse.individualId) {
    ids.push(h.headOfHouse.individualId);
  }
});

console.log('ids.length', ids.length);

function buildPhotoUrls(ids) {
  var maxLen = 2000;
  var urlPart1 = 'https://www.lds.org/mobiledirectory/services/ludrs/1.1/photo/url/';
  var urlPart2 = '/individual';
  var urls = [];
  var len = urlPart1.length + urlPart2.length;
  var batch = [];
  // starts at -1 because there is no comma on first element
  var batchLen = -1;

  if (!ids.length) {
    return urls;
  }

  ids.forEach(function (id) {
    id = id.toString();
    // NOTE +1 is for comma
    if (len + batchLen + id.length + 1 < maxLen) {
      batch.push(id);
      batchLen += id.length + 1;
    } else {
      console.log('batch.length', batch.length);
      urls.push(urlPart1 + batch.join(',') + urlPart2);
      batch = [];
      batchLen = -1;
    }
  });

  if (batch.length) {
    console.log(batch.length);
    urls.push(urlPart1 + batch.join(',') + urlPart2);
  }

  console.log('urls.length', urls.length);
  return urls;
}

buildPhotoUrls(ids).forEach(function (url) {
  console.log(
      'curl ' + url + ' \\'
    + '\n    --cookie-jar ./my-session.txt \\'
    + '\n    --cookie ./my-session.txt'
    )
});
```

It's a bit slow. Each request of ~170 ids takes about 6 seconds.

The good news is that you can run the requests in parallel, so you can still get the entire ward in about 6 seconds, which isn't too bad. 

6. Determining Gender
------------

You can't. Well, you can... but not in a reliable fashion.

**For MEMBERS in the ward**:

* https://www.lds.org/directory/services/ludrs/1.1/unit/roster/${LDS_HOME_WARD_ID}/ADULTS
* https://www.lds.org/directory/services/ludrs/1.1/unit/roster/${LDS_HOME_WARD_ID}/HIGH_PRIEST
* https://www.lds.org/directory/services/ludrs/1.1/unit/roster/${LDS_HOME_WARD_ID}/ELDER
* https://www.lds.org/directory/services/ludrs/1.1/unit/roster/${LDS_HOME_WARD_ID}/PRIEST
* https://www.lds.org/directory/services/ludrs/1.1/unit/roster/${LDS_HOME_WARD_ID}/TEACHER
* https://www.lds.org/directory/services/ludrs/1.1/unit/roster/${LDS_HOME_WARD_ID}/DEACON
* https://www.lds.org/directory/services/ludrs/1.1/unit/roster/${LDS_HOME_WARD_ID}/RELIEF_SOCIETY
* https://www.lds.org/directory/services/ludrs/1.1/unit/roster/${LDS_HOME_WARD_ID}/LAUREL
* https://www.lds.org/directory/services/ludrs/1.1/unit/roster/${LDS_HOME_WARD_ID}/MIA_MAID
* https://www.lds.org/directory/services/ludrs/1.1/unit/roster/${LDS_HOME_WARD_ID}/BEEHIVE

You **cannot** rely on `houseOfHouse`, nor **spouse**, because sometimes the active member is considered to be the headOfHouse and sometimes the husband is considered the head of house.

**For stake**:

* https://www.lds.org/mobiledirectory/services/v2/ldstools/member-detaillist-with-callings/${LDS_HOME_STAKE_ID}

You can **do a best guess** based on the calling, but although the urls above "work", they always return an empty array.

*workaround*

Normally you could iterate through all of the wards to find which ward the person belongs to, but since stake officials often serve outside of their home stake (especially YSA wards), this approach is probably not worth the effort of prefetching that data.

X. Other URLs
-------------

As stated previously, many of the URLs for the mobile API endpoints are listed here:
<https://tech.lds.org/mobile/ldstools/config.json>

Many of the web API endpoints are also listed here:
<https://github.com/LDSorg/lds.org-api-documentation>
