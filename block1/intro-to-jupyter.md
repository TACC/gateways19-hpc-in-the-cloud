Introduction to the Jupyter Notebook Environment and Tapis Token
===


Jupyter is an open source project that provides a webapp interface for writing code and documents. Throughout this tutorial, we will be using a Jupyter Notebook environment for development. 

### Starting up your Jupyter Notebook Environment

Your jupyter notebook server should be already running at `http://127.0.0.1:8888/?token=` (The one you got in the previous section after running the docker image). You can reach it by going to that location in a browser window.

Once you open a browser with your Jupyter environment, you should see something similar to this: 

<img src="../images/jupyter1.png" class="img-responsive" alt="Jupyter interface"> 


### Creating a Notebook

To create a new notebook for writing code, start by clicking 'New' in the upper right corner. From here, you will be able to choose what type of notebook you want. For this tutorial, we will be using Python 3. 

<img src="../images/jupyter2.png" alt="Jupyter Notebook">

Once you open a notebook, you can write and run python code. To execute a line of code, press `shift + Enter`. 


### Tapis v3 Token
As discussed in the introduction, we will use the official Tapis Python SDK for all of our interactions with the services. The Python SDK provides Python-native methods and objects for making HTTP requests and parsing HTTP responses to and from the Tapis API.
In order to do just about anything with Tapis, we will need to authenticate. Tapis makes heavy use of the notion of "tenants" in order to provide isolation for different projects. By setting the base_url variable, you indicate to the Tapis SDK which tenant you wish to interact with.

The "TACC tenant", with base URL "https://tacc.tapis.io", allows individuals to authenticate using any valid TACC account. For other tenants, the authentication rules could be different.
Authentication in the  "TACC" tenants use OAuth2 (again, this could be different in other tenants), but the Tapis Python SDK simplifies some of the complexity inherent in OAuth2 by providing some convenience functions for common use cases. For example, we are able to generate an access token using just our username and password via the convenience function “get_tokens()”. We do this below:
In your Jupyter notebook copy the block below and run it. This will give you a Tapis token to interact with the Tapis services
```
import datetime
import getpass
import os

username = getpass.getpass(prompt='Username: ', stream=None)
password = getpass.getpass(prompt='Password: ', stream=None)
base_url = 'https://tacc.tapis.io'

from tapipy.tapis import Tapis
#Create python Tapis client for user
client = Tapis(base_url= base_url, username=username, password=password)
# *** Tapis v3: Call to Tokens API
client.get_tokens()
# Print Tapis v3 token
client.access_token

```
