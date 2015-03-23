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

```
LDS_USERNAME='joesmith'
LDS_PASSPHRASE='super secret'
```

Then proceed to login:

```bash
curl https://signin.lds.org/login.html \
  --request POST \
  --location \
  --cookie ./my-session.txt \
  --cookie-jar ./my-session.txt \
  --data "username=${LDS_USERNAME}&password=${LDS_PASSPHRASE}"
```

The shorthand version of that command would be:

```bash
curl https://signin.lds.org/login.html \
  -X POST \
  -L \
  -b ./my-session.txt \
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

That will give you your `homeUnitNbr` and `individualId` which you will need for the next request

```bash
LDS_INDIVIDUAL_ID='1234567890'
LDS_HOME_WARD_ID='123456'
```

3. Ward Directory - Contacts and Callings
------------

**emails** and **phone numbers**

```
curl "https://www.lds.org/mobiledirectory/services/v2/ldstools/member-detaillist-with-callings/${LDS_HOME_WARD_ID}" \
  --cookie ./my-session.txt \
  --cookie-jar ./my-session.txt
```

It can be helpful to pipe this through `python -m json.tool` and save it to a file for easy reference:

```
curl "https://www.lds.org/mobiledirectory/services/v2/ldstools/member-detaillist-with-callings/${LDS_HOME_WARD_ID}" \
  --cookie ./my-session.txt \
  --cookie-jar ./my-session.txt \
  | python -m json.tool > ./my-ward.json
```

**NOTE**: In a YSA ward the Bishopric (external home ward) will be included, but their ids will be malformed (no household id).

4. Familiy and Individual Photos
------------

**Family Photo**

```
curl "https://lds.org/directory/services/ludrs/photo/url/${LDS_INDIVIDUAL_ID}/household" \
  --cookie ./my-session.txt \
  --cookie-jar ./my-session.txt
```

**NOTE**: The `houseOfHouseholdId` is *usually* the same as the `individualId`, but it's actually the church member of the household, which may be the wife or one of the children.

** Individual Photo**

```
curl "https://lds.org/directory/services/ludrs/photo/url/${LDS_INDIVIDUAL_ID}/individual" \
  --cookie ./my-session.txt \
  --cookie-jar ./my-session.txt
```

**NOTE**: I have no idea how you get the **Bishopric's Photos** in a **YSA ward**...

**Family Photo Directory**

This is the web API, not the mobile API, but it'll give you all of the family photo urls in one shot, which is a plus.

```
curl "https://www.lds.org/directory/services/ludrs/mem/wardDirectory/photos/${LDS_HOME_WARD_ID}" \
  --cookie ./my-session.txt \
  --cookie-jar ./my-session.txt
```

X. Other URLs
-------------

I'm not done with this tutorial yet, but many of the URLs are listed here:
<https://tech.lds.org/mobile/ldstools/config.json>

Many of them aro also listed here:
<https://github.com/LDSorg/lds.org-api-documentation>
