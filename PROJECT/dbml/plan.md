
- /api/v1/ 
- /health
- /search {both post and communites}
	- possible pagination here too ->>
	- search is fuzzy and returns filters are possible in the search strategy. 
	- post search is also fuzzy.
	- User chooses what to search before searching.
- /build_feed -> 
	- get the users top preferences like top k, by the tags. 
	- filter in the post table by the posts the user has not seen. 
	- get the k post and show the posts randomly and show it to the user. next time paginated the feed and show the next top k. 
IF THE POST IS SEEN. MARK IT IN THE post_views table. if the user like the post the weight +1 of  that posts tags for the user increses. if comment increase the weight of the post by +5. 

	
---
### **USER**

/user
POST    /signup    
POST    /login 
POST    /logout
GET     /profile
UPDATE  /update_profile 
PATCH   /update_password
	get       /get_otp  {for password reset only} {if possible only then do this}
DEL      /del_acc


/post
POST   /post/make 
	make the post add the tags 
		
DEL    /post/del
POST   /post/comment
%% POST %%  /post/comment/reply
	POST /mention {inside the comment only} 
PATCH   /post/edit
POST     /post/like
POST    /post/unlike

post     /tag/create_tag 
del       /tag/ del_tag

GET  /build_feed -> 
	1. keep the track of the top tags for the post. 
---

### **MODERATOR** 

user -> user_moderator_table -> [communities] dropdown of comminites.

dashboard features : 
/build_dashboard


GET       /community/post/new_posts 
DEL        /posts/del_post
PATCH   /posts/update_post
DEL       /posts/post/del_comment
POST     /user/rem 
POST     /user/mod {promote to moderator}

--- 

**ADMIN**

can do anything -> basically the feed shall be for the admin too : and we are gonna have the rights for the admin too. 

DEL      /post
DEL      /user 
DEL      /comment and if the comment has a lot of childs the message can be deleted. 
PATCH  /user/promote_and_demote {moderator->user}

---  

> ref of how the posts and the comments section might look like ? 

```
Post 101
│
├── Comment 1 (Great post!)
│     ├── Comment 2 (I agree)
│     │      └── Comment 3 (Same here)
│     └── Comment 4 (Thanks for sharing)
│
└── Comment 5 (Interesting idea)
      └── Comment 6 (Could you explain more?)
```


AI reponse on the api designing : 


TODO : april_fools

* ***REDIS STREAMS*** 
using both as a message broker and notification pubsub. 
so : 
1. if like : 
make event to update weights in the FeedService
	FeedService listens and run on the tags: for the user: 

2. do the cron job wala ki sync the databases on the update. 

3.  notification pubsub : notify about the event. 
4. 

