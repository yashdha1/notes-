server/
├── app/
│   ├── main.py
│   ├── config.py
│   ├── dependencies.py
│   ├── api/ -> all the routes stay here: 
│   │   ├── router.py
│   │   │   ├── auth.py
│   │   │   ├── users.py
│   │   │   └── items.py
│   │   └── __init__.py
│   ├── core/ 
│   │   ├── security.py
│   │   ├── config.py
│   │   └── logging.py
│   ├── models/ # schemas and models
│   │   ├── user.py
│   │   ├── item.py
│   │   └── __init__.py 
│   ├── services/
│   │   ├── user_service.py
│   │   ├── auth_service.py
│   │   └── item_service.py
│   ├── repo/
│   │   ├── user_crud.py
│   │   └── item_crud.py
│   ├── db/              
│   │   ├── session.py
│   │   ├── base.py
│   │   └── init_db.py
│   ├── utils/          # connection and strings
│   │   ├── email.py
│   │   └── helpers.py
│   └── middleware/
│       └── auth_middleware.py
├── requirements.txt
├── .env
└── README.md



client/ 
	node 
	npm 
	react + zustand for global stores
	keep the image of this light 

redis / 
		 cache 
		 authentication 

postgres / 
			storage 

dockerise the project 


#### Patterns: 
1. Composit pattern for the threaded messages. 
2. factory makes the messages. 
3. singleton for the db and rediss. 
4. Make advance searcher. [trie based or some other based in the DB MAYBE] 




- use uv 
	- ruff and the 
- pytest 
- logguru for the loggin purposes
- Server side events for the notifciation and the type shit: 
- 
