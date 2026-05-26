

/api/v1/ 

for the frontend : 
---
# user service /us

### auth  -> /auth

/refresh                          
/logout                          
/signup
/signin

### user ->  /user
/me
/password_reset
/{id}                    -> get the user with this id

---
# Project  /ps

### project  -> /project

/create                                 POST : create the projects
/list                                      GET : ALL THE PROJECT USER ASSOCIATED WITH : Authorised
/{project_id}                         GET : get all the details of the project with analytics. 
### ISSUES -> /{project_id}/issues

/                        GET : ALL THE issues of the project.
/create              POST : create issues.
?{filters}             GET : get issues by the filters. -> :All the filters: 
/{issue_id}         
	GET : Open the ISSUE  -> sub call the  /{issue_id}/comment/{lim} -> get all. with pagination.  
/{issue_id}         PATCH : change the status of the issue.  

### COMMENT -> /{project_id}/issues/{issue_id}/comment

/                                POST   -> create the comment for the issue_id.
		1. *User mentions krne ke liye...*
			1. ke pehele -> : onchange -> After @'username' 
					/{query}           **GET** all the users with this inits. 
				
/                                PATCH -> update the comment for the issue_id.
/del                           PATCH  -> update the body to deleted_comment. soft del
/


--- 

# NOTIFICATION /ns

### notification  -> /activity

/                                    GET : get all the notification.


--- 


# SEARCH -> /ss/search

/{query}                          GET:  in the meilisearch wala shit. 

--- 




1. Meilisearch image + service setup.
	1. search 
2. Messaging queue setup.
	1. post + notif + search 
3. SMTP POC.
	1. notif
4. Scaffolding.
	1. for both





### usage of the template in the email: 

```python
from app.models.template import issue_assigned

ctx = issue_assigned.IssueAssignedContext(
    assigned_to="jane",
    assigned_by="john",
    issue_title="Login page crashes on mobile",
    issue_body="Steps to reproduce...",
)

subject, html = issue_assigned.render(ctx)
await send_notification_email(SendEmailRequest(
    to_email="jane@example.com",
    subject=subject,
    body=html,
    is_html=True,
))
```


## TODO: 30 march 
 
```

# body of the 
{
  "total_issues": 150,
  "by_status": [{"name": "TODO", "count": 50}, {"name": "IN_PROGRESS", "count": 80}, ...],
  "by_priority": [{"name": "HIGH", "count": 20}, ...],
  "by_type": [{"name": "BUG", "count": 30}, {"name": "TASK", "count": 70}, ...],
  "combined": [
    {"status": "TODO", "priority": "HIGH", "issue_type": "BUG", "count": 5},
    ...
  ]
}
```

ALL OF THIS TODAY :->

--- 
--- 


**TODO**  
1. **Meilisearch**  -> index the create project and the create issue event. via the search topic. 
2. **Notification** -> added to *project* and assigned a *issue*. via the notification topic. 
3. **SMTP** notification service  ->in the notification service. 

Publishing and Subscribing via the kafka topics: 

1. publish (project) -> subscribing (search functionality) -> 
			1. new project created.
			2. new issue created.

2. publish (project) -> subscribing (notification functionality) -> 
			1. added to project. 
			2. new issue assigned.


DEPLOYMENT -> and moving the stack over to amazon aws SAAS. 


---

ticket req:-> 
	any changes suggested : 
	and testing that we need to do....
