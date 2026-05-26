pages : 

frontend : 

/login
/signup 
<navbar/>   search bar - profile - activity button 
activity_tab -> /activity 


/home
components :   

left_division   -> {filters : project creation} : {project list} : {project creation} : create_issue
right_division -> all the issues affiliated with the project. project_details issues list

page : 
	/project_name/issues/issue_id
	 all the details + comments
	change the status: 
		having all the details associated with the issue. 


popups : important for which we need to assign seperate time\

- create_project
- create_issue 



tech stack 


javascript + 
react lib for the components and pages building + 
zustand (global state manager) + 
tailwind for the css. 


#### plan and order: 

1. login and register page + /home basic outline + navbar
2. activity page + profile ops + left_select_bar + right_home page
3. right  page issues list and project details. 
4. create project page + create issue page
5. create issue viewer page. 
6. now the operations associated with the pages. 
----> connecting the pages and the backend.


