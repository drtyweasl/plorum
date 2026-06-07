Plorum Base URL: `https://plorum.net`

ALl requests that require authentication use a "Bearer token" in the `Authorization` header. Get the token by logging in first.
The `mobile` header is required to get a token to use, instead of a session cookie... Mobile doesn't like cookies, it thinks they're too gross.

---

## Authentication

### Login
```bash
curl -X POST https://plorum.net/api/auth/login \
  -H "Content-Type: application/json" \
  -H "X-Client-Type: mobile" \
  -d '{"username":"YOUR_USERNAME","password":"YOUR_PASSWORD"}'
```
**Response:**
```json
{
  "ok": true,
  "username": "weasl",
  "id": 1,
  "is_admin": false,
  "token": "abc123..."
}
```
Save the `token` value, you'll need it in every subsequent request.

---

### Register
```bash
curl -X POST https://plorum.net/api/auth/register \
  -H "Content-Type: application/json" \
  -H "X-Client-Type: mobile" \
  -d '{"username":"YOUR_USERNAME","email":"YOUR_EMAIL","password":"YOUR_PASSWORD"}'
```
**Response:**
```json
{ "ok": true, "username": "yourname", "id": 2, "token": "abc123..." }
```

---

### Logout
Revokes your mobile token.
```bash
curl -X POST https://plorum.net/api/auth/logout \
  -H "Authorization: Bearer TOKEN"
```

---

## Posts

### Get all posts (no pagination)
```bash
curl "https://plorum.net/api/posts?limit=all"
```

### Get posts sorted by new
```bash
curl "https://plorum.net/api/posts?limit=all&sort=new"
```
Available sorts: `hot`, `new`, `top`, `rising`, `controversial`

### Get posts from a specific plorum
```bash
curl "https://plorum.net/api/posts?limit=all&channel=offtopic"
```

### Get a specific number of posts
```bash
curl "https://plorum.net/api/posts?limit=50"
```

### Get a single post
```bash
curl "https://plorum.net/api/posts/123"
```

### Create a post
```bash
curl -X POST https://plorum.net/api/posts \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer TOKEN" \
  -d '{"title":"My post","body":"Hello!","plorum":"offtopic","type":"text"}'
```
Available types: `text`, `link`, `image`

For a link post, add `"url":"https://example.com"`.

### Delete your own post
```bash
curl -X POST https://plorum.net/api/posts/123/delete \
  -H "Authorization: Bearer TOKEN"
```

### Vote on a post
```bash
curl -X POST https://plorum.net/api/posts/123/vote \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer TOKEN" \
  -d '{"dir":"up"}'
```
`dir` is either `"up"` or `"down"`.

### Report a post
```bash
curl -X POST https://plorum.net/api/posts/123/report \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer TOKEN" \
  -d '{"reason":"Spam"}'
```

---

## Comments

### Get comments for a post
```bash
curl "https://plorum.net/api/posts/123/comments"
```

### Post a comment
```bash
curl -X POST https://plorum.net/api/posts/123/comments \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer TOKEN" \
  -d '{"body":"This comment is horrible."}'
```

### Reply to a comment (replace 456 with parent comment ID)
```bash
curl -X POST https://plorum.net/api/posts/123/comments \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer TOKEN" \
  -d '{"body":"This reply to a comment is horrible.","parentId":456}'
```

### Vote on a comment
```bash
curl -X POST https://plorum.net/api/comments/456/vote \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer TOKEN" \
  -d '{"dir":"up"}'
```

---

## Plorums

### Get all plorums
```bash
curl "https://plorum.net/api/plorums"
```

### Get a specific plorum
```bash
curl "https://plorum.net/api/plorums/offtopic"
```

### Subscribe / unsubscribe (toggles)
```bash
curl -X POST https://plorum.net/api/plorums/1/subscribe \
  -H "Authorization: Bearer TOKEN"
```
Use the plorum's number `id` from the plorums list.

---

## User

### Get your own profile
```bash
curl "https://plorum.net/api/me" \
  -H "Authorization: Bearer TOKEN"
```

### Get any user's profile + recent posts
```bash
curl "https://plorum.net/api/users/11friend"
```
Returns public profile info and their 20 most recent posts.

### Search
```bash
curl "https://plorum.net/api/search?q=11friend"
```
Returns matching posts, users, and Plorums.

---

## Notifications

### Get notifications
```bash
curl "https://plorum.net/api/notifications" \
  -H "Authorization: Bearer TOKEN"
```

### Mark all notifications as read
```bash
curl -X POST https://plorum.net/api/notifications/read \
  -H "Authorization: Bearer TOKEN"
```

---



Avatars can be added from Gravatar using:
```url
https://www.gravatar.com/avatar/{md5(email)}?s=SIZE&d=identicon
```
---

## To upload an image (and then use it in a post), do the following:

### Step one (this will upload the photo to the server):
```bash
curl -X POST https://plorum.net/api/upload/image \
  -H "Authorization: Bearer TOKEN" \
  -F "image=@/path/to/photo.jpg"\
```

this will return:
```json
{ "url": "/uploads/abc123.jpg" }
```

## Step two (use the image in a post):
```bash
curl -X POST https://plorum.net/api/posts \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer TOKEN" \
  -d '{"title":"My image post","type":"image","image_url":"/uploads/abc123.jpg","plorum":"offtopic"}'
```

### Basically, upload the image, grab the returned URL from step one, then use the URL in `image_url` on step two!

---

### Changing avatar via upload

```bash
curl -X POST https://plorum.net/api/upload/avatar \
  -H "Authorization: Bearer TOKEN" \
  -F "avatar=@/path/to/photo.jpg"
```
## Notes: Automatically disables Gravatar when uploaded.
Returns 
```json
{ "url": "/uploads/<filename>" }
```
---

## Tips

- Replace `TOKEN` with the token string returned by login/register.
- Replace `123` / `456` with actual post/comment IDs from the feed.
- Replace `offtopic` with actual Plorum URL name ids from `/api/plorums`.
- All responses are JSON.
- HTTP errors return `{"error":"message"}` with a status code.
- Don't forget that mobile doesn't like cookies. Thats very important.
