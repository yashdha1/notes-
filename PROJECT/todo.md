## **Mini Project: Advanced Real-Time Discussion Forum**

This project focuses on developing a scalable and interactive discussion forum that allows real-time communication and collaboration among users. Participants will apply advanced concepts to design and implement the system.

**Key Features**:
- **User Authentication**: Secure registration and login using JWT.
- **Real-Time Messaging**: Enable instant communication with WebSocket technology.
- **Threaded Discussions**: Support nested comments for organized topic discussions.
- **Search Functionality**: Implement search filters to quickly locate threads and comments.
- **Notifications**: Provide real-time updates for new messages, replies, and mentions.
- **Role Management**: Define user roles such as admin, moderator, and member for permissions.

**1. User Management**
- The system shall allow users to register using an email and password.
- The system shall allow users to log in using JWT.
- The system shall issue an access token upon successful login.
- The system shall allow users to update their profile information (name, avatar, bio).
- The system shall allow users to reset their password.

**2. Role & Permission Management**
- The system shall support roles: **Admin**, **Moderator**, and **Member**.
- Admin users shall be able to promote/demote user roles.
- Members can create/edit/delete only their own posts and comments.
- Moderators can delete or edit any post or comment.
- Admins can manage users, roles.

**3. Threads & comments**
- The system shall allow authenticated users to create new discussion threads with a title and description.
- The system shall display a list of all threads with pagination. Allow the fetching of certain amt of comments only and once the user clicks more replies fetch more.
- The system shall display all comments and nested comments under a specific thread.
- The system shall support multi‑level nested comments.
- A user can like a thread or comment only once.
- When a user likes a thread, all connected clients should see the updated like count immediately (WebSocket update).
- The system shall allow users to edit or delete their own posts.
- Moderators/Admins shall be able to modify or delete any post.


**4. Real‑Time Communication**
- The system shall deliver new posts and comments instantly to all connected clients using WebSockets.
- The system shall update thread view automatically without page refresh.


**5. Search Functionality**
**Search Threads**
- The system shall allow users to search threads by keywords. 


**6. Notification System**
The system shall send real-time notifications when:
- A user receives a reply to their post.
- A user is mentioned using @username.
- **Admin Dashboard:** Manage users, roles,.
- **Moderator Dashboard:** Tools to monitor, review, and moderate threads, posts, and user activity.
- **Member Dashboard:** Simple interface to create threads, post comments, like content, and view notifications.

dockerise from the start of this application : 

---

**BACKEND**

dev tools : 
		uv -> everything else would be figured on the fly...
smtp:
SocketIO - WebSockets:
CLOUDFLARE for the CDN:
JWT:
Composite pattern for the threaded messages:  
Pydantic: 
redis

Search in the DB -> caching dalna hai isme:


user 
post 
messaging between the user / dont think about this dawg. 
search in the post all the post 



> Create story points make the frontend first, and keep the api wala scene concurent. 
then concurrrently start working on the backend wala scene. 


Pagination in the comments : tech for now: 

1. by default fetch top 100 comments in the post by default. 
	1. if clicked on the replies: fetch the current parent root 10 nested comments
		1. if the comments next comments is clicked fetch the current parent comment nested 10 comment for this post. 
at the end of each comment wala scene there will be a sense of fetching next 'n' comments. simple? 
like in the instagram.

- uvicorn for dev 
- uvicorn + gunicorn for production enviorment. 

---

**FRONTEND :**  


TEMPLATE :  https://github.com/vintasoftware/nextjs-fastapi-template.git _> should be the cleanest one in the bunch -> 

--- 

- complete the feed microservice and start working on the next application : 
- meri post pe notyification :  
- check, redis streams events additions, procced accordingly. Usko add krneka project mein. 


make the frontend.
1. Do the feed service. 
2. fix the project structure. repo, handlers.
3. and start the next application anyhow. does not matter. die nigga.



## **BACKEND LEFT** 

before all this fix the strcure of the project.
1. redis pubsub : like complete. 
2. rooms wala. 
3. message broker.
4. cron jobs. 
5. then start fine tuning. 
6. read the requirements again and again. 
7. 