# Forum API working documentation

# TODO
- How should authentication be implemented? What fields will be used, and sent in what format? (HTTP request headers?)
    - And how should logging in and logging out work? What will the server send and what will the client store?
- How should POST messages be formatted? (JSON in request body?)
- Pagination is currently done by having the client send a page number and a per-page limit. Should this have been implemented some other way?
- How should we deal with updates? - Suppose the server updated (and API was changed) but someone's browser didn't refresh yet, or had some really stubborn cache. What should be done to minimize the impact of incompatible API formats between the mismatching server/client versions?

## General requirements
- All APIs should be (first) implemented to return data in JSON format, if a response is needed.
- All APIs should implement an [Optional] [Non-essential] URL parameter `format` of type String. When not present or has an invalid value, should default to `json`. Before any other formats are implemented, this parameter does nothing, but the server should be prepared for this to possibly exist.
- [Optional] fields may be omitted by the client.
- [Non-essential] fields are not essential for the minimum viable product. Should time be not enough for this feature to be implemented, this field would do nothing, but the server should expect this field to possibly exist.
    - In case there is any confusion: [Optional] fields **have to be implemented**, just the client might omit the field in the requests. [Non-essential] fields **don't need the functionality be implemented** for the time being, but the client might send this field anyway, in which case it simply does nothing.
- On success, the response body should return a table with two fields, `success`=true, and `data` containing the requested data. On failure, in addition to a 4xx or 5xx HTTP status code, the response body should also contain a table with two fields, `success`=false, and `error`, which is a string that might be displayed to the end user (can be an empty string). Since it might be displayed to the end user, always use human-readable sentences, and avoid giving away information that might become a security risk.
- Beware of XSS! (Who should do the sanitizing? The server, or the client, or both?)

## Defined APIs

### GET `/api/news/newsList`
Obtain a list of "hot news".

URL parameters:
- [Optional] [Non-essential] String `keyword`: If present, should return news that are relevant to this keyword.
- [Optional] [Non-essential] Int `board`: If present, should return news that are relevant to this board.
- [Optional] [Non-essential] Int `post`: If present, should return news that are relevant to this post.
- [Optional] Int `number`: Number of entries to return. Default to 10.

Response data format:
- Int `number`: the number of entries actually returned. Might be lower than requested, should there be no enough articles.
- Array `articles`: an array of tables, each representing a news article, with the following fields:
    - String `title`: title of the article.
	- String `thumbnail`: URL to a thumbnail image for this article. Should a thumbnail be unavailable, this field should be an empty string.
	- String `summary`: a summary of the article to display under the title. Should the source not provide a summary, the first 150 characters of the content of the article should suffice.
	- String `url`: URL to the article.

### GET `/api/boards`
Obtain a list of boards.

URL parameters:
- [Optional] [Non-essential] String `sort`: How the returned results should be ordered.
- [Optional] [Non-essential] String `search`: Search keyword for searching boards.
- [Optional] Int `limit`: how many results to return. Default to 20.
- [Optional] Int `page`: which page to return. 1-indexed, default to 1. Should client request a number larger than the page number of the last page, the server should send the last page instead.

Response data format:
- Int `number`: number of results returned in this request.
- Int `page`: Actual page number being sent back.
- Int `totalpages`: Total number of pages.
- Array `boards`: an array of tables, each representing a board, with the following fields:
    - String `name`: name of the board
	- Int `id`: ID of the board

**TODO**: Might there be additional information that needs to be sent to the client, say any additional icons like "official", "featured" or "pinned"?

### GET `/api/board/info`
Obtain metadata about a board.

Alias: `/api/board/<boardID>/info`

URL parameters:
- Int `boardid`

Response data format:
- **TODO**: This section should return certain metadata about this board, eg. background image, description, side bar (like in Reddit or Tieba), etc. What data should go here?

### GET `/api/board/posts`
Obtain list of posts in a board.

Alias: `/api/board/<boardID>/posts`

URL parameters:
- Int `boardid`
- [Optional] [Non-essential] String `sort`: How the returned results should be ordered.
- [Optional] Int `limit`: how many results to return. Default to 20.
- [Optional] Int `page`: which page to return. 1-indexed, default to 1. Should client request a number larger than the page number of the last page, the server should send the last page instead.

Response data format:
- Int `number`: number of results returned in this request.
- Int `page`: Actual page number being sent back.
- Int `totalpages`: Total number of pages.
- Array `posts`: an array of tables, each representing a post, with the following fields:
    - String `title`: title of the post
	- Int `postid`: ID of the post
	- String `author`: name of the author of the post
	- Int `authorid`: UserID of the author
	- String `summary`: The first 100 characters of the post, in plain text, with line breaks, formatting marks, media, and extra whitespaces removed.
	- Int `upvotes`: the number of upvotes (likes) on this post
	- Int `downvotes`: the number of downvotes (dislikes) on this post
	- Int `timecreated`: a UNIX timestamp of when this post was made.
	- Boolean `userupvoted`: Should a user be logged in, this boolean represent whether this user has already upvoted this post.
	- Boolean `userdownvoted`: Should a user be logged in, this boolean represent whether this user has already downvoted this post.
	- Array `tags`: an array of strings, containing tags (Reddit: flares) on this post, for example, "featured", "pinned", etc.

**TODO**: When the user eventually clicks on a post, some info about that particular post might need to be sent again. Do we accept the repetition, or should we implement something that reduces repeated data transfers? (Also note that a user might skip the board page and directly access the post page by using a URL.)

### GET `/api/post/content`
Obtain information and content about a post - this does not include any replies under this post.

Alias: `/api/post/<postID>/content`

URL parameters:
- Int `postid`

Response data format (differences from `/api/board/posts` highlighted in **bold**):
- String `title`: title of the post
- Int `postid`: ID of the post
- String `author`: name of the author of the post
- **String `profileimage`**: URL to the profile image of the author
- Int `authorid`: UserID of the author
- **String `content`**: Content of the post
- Int `upvotes`: the number of upvotes (likes) on this post
- Int `downvotes`: the number of downvotes (dislikes) on this post
- Int `timecreated`: a UNIX timestamp of when this post was made.
- Boolean `userupvoted`: Should a user be logged in, this boolean represent whether this user has already upvoted this post.
- Boolean `userdownvoted`: Should a user be logged in, this boolean represent whether this user has already downvoted this post.
- Array `tags`: an array of strings, containing tags (Reddit: flares) on this post, for example, "featured", "pinned", etc.

### GET `/api/post/replies`
Obtain replies under a post.

Alias: `/api/post/<postID>/replies`

URL parameters:
- Int `postid`
- [Optional] [Non-essential] String `sort`: How the returned results should be ordered.
- [Optional] Int `limit`: how many results to return. Default to 20.
- [Optional] Int `page`: which page to return. 1-indexed, default to 1. Should client request a number larger than the page number of the last page, the server should send the last page instead.

Response data format:
- Int `number`: number of results returned in this request.
- Int `page`: Actual page number being sent back.
- Int `totalpages`: Total number of pages.
- Array `replies`: an array of tables, each representing a reply, with the following fields:
	- Int `replyid`: ID of the reply
	- String `author`: name of the author of the reply
	- Int `authorid`: UserID of the author
	- String `profileimage`: URL to the profile image of this user
	- String `content`: contents of the reply
	- Int `upvotes`: the number of upvotes (likes) on this post
	- Int `downvotes`: the number of downvotes (dislikes) on this post
	- Int `timecreated`: a UNIX timestamp of when this post was made.
	- Boolean `userupvoted`: Should a user be logged in, this boolean represent whether this user has already upvoted this reply.
	- Boolean `userdownvoted`: Should a user be logged in, this boolean represent whether this user has already downvoted this reply.

### GET `/api/users/profile`
Obtain a user's profile.

**TODO**: I don't think this is needed in a minimum viable product. Should we define this API anyway?

### POST `/api/createpost`
Create a post.

Should the user be not logged in or lack permissions (eg, ongoing ban), a 403 should be returned.

Request body format:
- Int `boardid`: this post is posted under which board
- String `content`

Response data format:
- Int `postid`: the postID of the newly created post.

### POST `/api/createreply`
Create a reply.

Should the user be not logged in or lack permissions (eg, ongoing ban), a 403 should be returned.

Request body format:
- Int `postid`: this reply is replying to which post
- String `content`

Response data format:
- Int `replyid`: the replyID of the newly created reply.
- String `author`: name of the author of the reply
- Int `authorid`: UserID of the author
- String `profileimage`: URL to the profile image of this user
- String `content`: contents of the reply, sanitized
- Int `timecreated`: a UNIX timestamp of when this post was made.

## Wish list
- [X] GET "hot news"
- [X] GET main page (list of boards)
- [ ] POST login
- [ ] POST register
- [ ] GET/POST? logout
- [X] GET board (post list)
- [X] GET post (post content and replies)
- [X] GET search board (implemented as a part of "list of boards")
- [X] POST post
- [X] POST comment/reply
- [ ] POST delete post
- [ ] POST delete reply
- [ ] POST like/dislike/unlike/undislike post
- [ ] POST like/dislike/unlike/undislike comment
- [X] GET user profile
- [ ] POST edit user profile
- [ ] POST report
- [ ] POST delete account
- [ ] POST manage board?
- [ ] POST manage user ban?
