READ THIS FIRST
===============

You should not need to access the LDS.org API directly with cURL.

Consider using using the **OAuth2 API at <https://lds.io>** instead.

This tutorial is provided for the sake of prototyping and exploring, not for production use.

LDS.org with cURL
=================

**NOTE**: Some APIs have a trailing slash. Some don't. **It matters which is which**.

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

That will give you your `individualId` which you will need for the next request

```bash
LDS_INDIVIDUAL_ID='1234567890'
LDS_HOME_WARD_ID='123456'
```

3. Your Ward
------------

```
curl "https://www.lds.org/mobiledirectory/services/v2/ldstools/member-detaillist-with-callings/${LDS_HOME_WARD_ID}" \
  --cookie ./my-session.txt \
  --cookie-jar ./my-session.txt
```

X. Other URLs
-------------

I'm not done with this tutorial yet, but many of the URLs are listed here:
<https://tech.lds.org/mobile/ldstools/config.json>

Many of them aro also listed here:
<https://github.com/LDSorg/lds.org-api-documentation>
