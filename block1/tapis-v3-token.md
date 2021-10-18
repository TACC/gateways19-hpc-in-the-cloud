Tapis v3 Token
===

As discussed in the introduction, we will use the official Tapis Python SDK for all of our interactions with the services. The Python SDK provides Python-native methods and objects for making HTTP requests and parsing HTTP responses to and from the Tapis API.
In order to do just about anything with Tapis, we will need to authenticate. Tapis makes heavy use of the notion of "tenants" in order to provide isolation for different projects. By setting the base_url variable, you indicate to the Tapis SDK which tenant you wish to interact with.

The TACC tenant, with base URL https://tacc.tapis.io, allows individuals to authenticate using any valid TACC account. For other tenants, the authentication rules could be different.
Authentication in the TACC tenants use OAuth2 (again, this could be different in other tenants), but the Tapis Python SDK simplifies some of the complexity inherent in OAuth2 by providing some convenience functions for common use cases. For example, we are able to generate an access token using just our username and password via the convenience function get_tokens(). We do this below:
In your Jupyter notebook copy the blocks below and run them or just run them directly from the example notebook. You will get a Tapis v3 token to interact with the Tapis services.

```
#Enter training account info
import getpass

tenant = 'tacc'
base_url = 'https://' + tenant + '.tapis.io'

username = input('Username: ')
password = getpass.getpass(prompt='Password: ', stream=None)
host = input('Host: ')

```

```
#Generate access token
from tapipy.tapis import Tapis
#Create python Tapis client for user
client = Tapis(base_url= base_url, username=username, password=password)
# *** Tapis v3: Call to Tokens API
client.get_tokens()
# Print Tapis v3 token
client.access_token

```