
Search_wala_system : -> 


1. **meilisearch**+ light wrapper around it. with SQS.  docker setup -> light fastapi wrapper around this. -> SQS publish and subscribe setup. that would consume the event from there and index them.  (fastpapi + dockerise) + (fastapi consumer). search. index. 
	1. dependency -> publisher from the project service. on the SQS buses. 

2. **Integration** :- 
		dependency -> API + tested + pages frontend should be completed. 
		integrations in the project in the format of the: 
		
	1. login+signup+logout-> backend apis _>  creteria users are able to perform these action. 
	2. navbar + search + activity page integrations -> backend apis. 
	3. left page _>  create projects  and part of project. filters.  -> integrating with backend apis. 
	4. right page _> project details + authorised paths in this + and should fetch all the issues associated with this. 
	5. **open issue** _> issue page with all the comments, comment making api. interat with the issues change state + stuff
	6. create issue _>  
	7. create project + create epic _>  



## FORMAT : 
```
---
epic: "name"
title: "ticket_title"
status: in-progress
assignee: ""
created: 2026-04-23
---

# Ticket Title

## 1. Vision
> One or two sentences. What are we building and why.
> description short 

---

## 2. Goals & Success Metrics

| Goal | Success Metric |
|------|----------------|
| ...  | ...            |

---

## 4. Functional Requirements (MoSCoW)

### Must Have (M)
- ...

### Should Have (S)
- ...

### Could Have (C)
- ...

### Won't Have (W)
- ...

---

## 5. Non-Functional Requirements

| Category    | Requirement |
|-------------|-------------|
| Performance | ...         |
| Security    | ...         |
| Code Quality| ...         |
| State       | ...         |
| Routing     | ...         |

---

## 6. Dependencies

- **Runtime:** ...
- **Dev / Build:** ...
- **Backend:** ...
- functional behind that we are in the sause. 

---

## 8. Acceptance Criteria

- [ ] ...
- [ ] ...
- standard of code (name of function,testcases, documented (in the docstring awa seperated group document.))

---

## 9. Build Order -> if there are any dependencies we can manage them here. 

| Stage | Scope |
|-------|-------|
| S0    | ...   |
| S1    | ...   |

---

## 11. Out of Scope (v1) -> Kya nahi karna hai

- ...
- ...
``` 
  
aggegate

Analytics 

issues filter by the status + success + failed + progress + timeline completion + overdue + members : 