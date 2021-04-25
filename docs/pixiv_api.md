# Pixiv API ([source code](../pxvpy3/pixiv_api.py))

- [Authentication](#authentication)
    - [Login](#login)
    - [Refresh](#refresh)
- [User](#user)
    - [User Detail](#user-detail)

## Authentication

Get a user token and create a client for future HTTP requests <br>

### Login

([source code](../pxvpy3/pixiv_api.py#L213))

Example:

```py
from pxvpy3.pixiv_api import PixivClient

# will receive email
with PixivClient.login('email', 'password') as client:
    print(client.token.refresh_token)
```

- Caution: Loggin in with email and password will cause Pixiv to send an email saying:
  > [pixiv] 新しいログインがありました (場所: {country}) <br>
  [pixiv] New login to pixiv from {country} <br>
- To avoid this, simply login with refresh token obtained from `client.token.refresh_token`

### Refresh

([source code](../pxvpy3/pixiv_api.py#L229))
Example:

```py
from pxvpy3.pixiv_api import PixivClient

# will not receive email
with PixivClient.refresh('refresh token') as client:
    # HTTP requests
    pass
```

## User

Send GET requests at `/v1/user/*`

- Methods that take the `callback` argument will keep executing `callback(page)` until reach the end or rate limit<br>
  In order to deal with rate limit, set the `request_handler` field:
    ```py
    import time
    
    from requests.models import Response
    
    from pxvpy3.pixiv_api import PixivClient
    
    
    def request(response: Response, is_successful: bool) -> bool:
        if is_successful:
            return True
        elif response.status_code == 403:  # forbidden
            print('Calm down for 5 minutes')
            time.sleep(300)
            # send the request again
            return False
        # will raise error
    
    
    with PixivClient.refresh('refresh token') as client:
        client.request_handler = request
        # HTTP requests
    ```

### User Detail

([source code](../pxvpy3/pixiv_api.py#L265))
Example:

```py
from pxvpy3.pixiv_api import PixivClient

with PixivClient.refresh('refresh token') as client:
    # get detail of myself
    myself = client.get_user_detail()
    # get detail of other users
    sakimori = client.get_user_detail(211515)
```

### User Illustrations

([source code](../pxvpy3/pixiv_api.py#L274))
Example:

```py
from pxvpy3.pixiv_api import PixivClient, IllustsPage


def callback(page: IllustsPage):
    """
    Print every illustration ID in a page
    """
    for illust in page.illusts:
        print(illust.id)


with PixivClient.refresh('refresh token') as client:
    # this will call `callback` repeatedly until the end or rate limit
    client.get_user_illusts(callback)
```