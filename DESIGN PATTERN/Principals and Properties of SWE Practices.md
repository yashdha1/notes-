
THEORY :  Source : 
[zedr/clean-code-python: :bathtub: Clean Code concepts adapted for Python](https://github.com/zedr/clean-code-python?tab=readme-ov-file#introduction)
[heykarimoff/solid.python: SOLID Principles explained in Python with examples.](https://github.com/heykarimoff/solid.python/tree/master)

challenges and other references : 
https://github.com/ashishps1/awesome-low-level-design?tab=readme-ov-file 


**SOLID**

```
* S  -> SRP  _ (ek class ek responiblity)
* O ->  Open Close _ (modify mat karro aur kuch dalna hai dalskte)
* L  ->  LSP  _ (child ke jagah pe parent ko atte ana chahiye ie. child should have every property of parent)
* I ->  ISP _  (bacche ke pass uss functionality ka access nahi hona chahiye jo vo istemaal nahi karega)
* D -> DIP _ (ie. Duck typing more or less.)
```

# 1. Open/Close Principal

> **"Software entities should be open for extension, but closed for modification."** — Bertrand Meyer / Robert C. Martin

---
#### The Core Idea
- **Open for extension** → you can add new behavior
- **Closed for modification** → you don't touch existing, tested code
---
#####  Violating OCP

```python
class DiscountCalculator:
    def calculate(self, customer_type: str, price: float) -> float:
        if customer_type == "regular":
            return price
        elif customer_type == "vip":
            return price * 0.8
        elif customer_type == "student":
            return price * 0.9
        # Adding anything here mofdifys the preexixting shit: 
        # bad practice : violation of the opencloseprincipal
```

Every time a new customer type is added, you must **modify** `DiscountCalculator` — risking bugs in existing logic.

---
#### Following OCP

Use **abstraction** (ABC or Protocol) so new behavior is added by _extension_, not _modification_.

```python
from abc import ABC, abstractmethod

# 1. Define a stable abstraction
class Discount(ABC):
    @abstractmethod
    def apply(self, price: float) -> float:
        pass

# 2. Concrete implementations (extensions)
class RegularDiscount(Discount):
    def apply(self, price: float) -> float:
        return price

class VIPDiscount(Discount):
    def apply(self, price: float) -> float:
        return price * 0.8

class StudentDiscount(Discount):
    def apply(self, price: float) -> float:
        return price * 0.9

# 3. Calculator is CLOSED — never needs to change
class DiscountCalculator:
    def calculate(self, discount: Discount, price: float) -> float:
        return discount.apply(price)

# Adding a new type = just a new class, nothing else touched ✅
class FlashSaleDiscount(Discount):
    def apply(self, price: float) -> float:
        return price * 0.5
```

```python
calc = DiscountCalculator()
print(calc.calculate(VIPDiscount(), 100))        # 80.0
print(calc.calculate(FlashSaleDiscount(), 100))  # 50.0
```
##### Another Example — Shape Area

```python
from abc import ABC, abstractmethod
import math

class Shape(ABC):
    @abstractmethod
    def area(self) -> float:
        pass

class Circle(Shape):
    def __init__(self, radius: float):
        self.radius = radius
    def area(self) -> float:
        return math.pi * self.radius ** 2

class Rectangle(Shape):
    def __init__(self, w: float, h: float):
        self.w, self.h = w, h
    def area(self) -> float:
        return self.w * self.h

# Closed — works for ANY future shape automatically
def total_area(shapes: list[Shape]) -> float:
    return sum(s.area() for s in shapes)

class Triangle(Shape):
    def __init__(self, base: float, height: float):
        self.base, self.height = base, height
    def area(self) -> float:
        return 0.5 * self.base * self.height
```
##### Using `Protocol` (Python 3.8+) — no inheritance needed

```python
from typing import Protocol

class Serializable(Protocol):
    def serialize(self) -> str: ...

class JSONUser:
    def serialize(self) -> str:
        return '{"type": "json"}'

class XMLUser:
    def serialize(self) -> str:
        return '<type>xml</type>'

def export(item: Serializable) -> None:
    print(item.serialize())

# Add new formats without touching export() ✅
```
##### Summary

|Concept|Meaning|
|---|---|
|**Closed for modification**|Core logic is stable & tested — don't touch it|
|**Open for extension**|Add new classes/modules to introduce new behavior|
|**Key tool**|Abstract base classes, Protocols, dependency injection|

 The **if/elif chain smell** is the most common OCP violation — when you see one, ask: _"can I replace this with polymorphism?"_

---

# 2. Single Responsibility Principle (SRP) in Python

> **"A class should have only one reason to change."** — Robert C. Martin

The Core Idea
Each class/function should own **one** well-defined job. If you can describe a class with the word **"and"**, it's doing too much.

* E1 - Breakdown -> Violating SRP

```python
from importlib import metadata

class VersionCommentElement:
    def get_version(self) -> str:   
        return metadata.version("pip")

    def render(self) -> None: 
        print(f'<!-- Version: {self.get_version()} -->')
```

**Two reasons to change:**
- The way version is _fetched_ changes (e.g. read from a config file instead)
- The way HTML is _rendered_ changes (e.g. return string instead of print)

Changing either one risks breaking the other.
### Following SRP

```python
from importlib import metadata

# Responsibility 1: Data retrieval — lives here only
def get_version(pkg_name: str) -> str:
    return metadata.version(pkg_name)

# Responsibility 2: Rendering — lives here only
class VersionCommentElement:
    def __init__(self, version: str):
        self.version = version

    def render(self) -> None:
        print(f'<!-- Version: {self.version} -->')

# Wired together at the call site
VersionCommentElement(get_version("pip")).render()
```

Now each piece has **exactly one reason to change**.
#### Another Example — User Management
Violating SRP

```python
class User:
    def __init__(self, name: str, email: str):
        self.name = name
        self.email = email

    def save_to_db(self):          # Responsibility: Persistence
        print(f"Saving {self.name} to database...")

    def send_welcome_email(self):  # Responsibility: Notifications
        print(f"Sending welcome email to {self.email}...")

    def format_display(self) -> str:  # Responsibility: Presentation
        return f"User({self.name}, {self.email})"
```

This class changes if the DB changes, if email logic changes, or if display format changes.
#### Following SRP
```python
class User:
    """Only knows about user data."""
    def __init__(self, name: str, email: str):
        self.name = name
        self.email = email

class UserRepository:
    """Only knows about persistence."""
    def save(self, user: User):
        print(f"Saving {user.name} to database...")
        
class EmailService:
    """Only knows about notifications."""
    def send_welcome(self, user: User):
        print(f"Sending welcome email to {user.email}...")

class UserFormatter:
    """Only knows about presentation."""
    def format(self, user: User) -> str:
        return f"User({user.name}, {user.email})"
```

```python
user = User("Alice", "alice@example.com")

UserRepository().save(user)
EmailService().send_welcome(user)
print(UserFormatter().format(user))
```
#### A Useful Smell Test

|Signal|What it suggests|
|---|---|
|Class name contains **"And"**|Two responsibilities|
|Method does X **then** sends an email|Mixed concerns|
|A change in DB breaks email logic|Tight coupling = SRP violation|
|Hard to unit test one part without the other|Responsibilities are tangled|
 Key Takeaways

- **One class → one job** → one reason to change
- Separation makes each piece **independently testable** and **reusable** (`get_version()` can now serve anywhere)
- SRP doesn't mean _one method per class_ — it means one **cohesive area of responsibility**
- The glue between separated pieces lives at the **call site**, not inside the classes themselves.
---
# 2.5 Mixin Classes in Python

> A **Mixin** is a class that provides methods to other classes through inheritance, but is **not meant to stand alone**.

The Core Idea -> acts like a plugin in between the class reponsiblities. 
- Mixins are like **plug-in capabilities** you attach to a class
- They avoid code duplication without deep inheritance chains
- A mixin does **one small thing** and does it well
#### Simple Example

```python
class Logger: 
    """MIXIN : Adds logging capability to any class"""
    def log(self, message: str) -> None:
        print(f"[{self.__class__.__name__}] {message}")


class Serializer:
    """MIXIN : Adds JSON serialization to any class"""
    def to_dict(self) -> dict:
        return self.__dict__

# Plug in only what you need
class User(Logger, Serializer):
    def __init__(self, name: str, email: str):
        self.name = name
        self.email = email

    def save(self):
        self.log(f"Saving user {self.name}...")  # from LogMixin


class Product(Logger):          # only needs logging
    def __init__(self, name: str, price: float):
        self.name = name
        self.price = price

    def update_price(self, new_price: float):
        self.log(f"Price changed: {self.price} → {new_price}")
        self.price = new_price
```

```python
user = User("Alice", "alice@example.com")
user.save()           # [User] Saving user Alice...
print(user.to_dict()) # {'name': 'Alice', 'email': 'alice@example.com'}

p = Product("Laptop", 999)
p.update_price(899)   # [Product] Price changed: 999 → 899
```
#### How Mixins Differ from Regular Inheritance

```python
# Regular inheritance — "is-a" relationship
class Animal:
    pass
class Dog(Animal):   # Dog IS AN Animal makes sense
    pass

# Mixin — "can-do" relationship  
class Dog(LogMixin):  # Dog CAN LOG — behaviour bolt-on..
    pass
```

### so diffrence is? 
1. Regular class inheritance -> *is - a* relationship       [Version Representation]
2. Regular class composition -> *has - a* relationship  [Component Representation]
3. Mixins reprentations  -> *can-do* relationship         [Behavorial Representation]

||Regular Base Class|Mixin|
|---|---|---|
|**Purpose**|Define what a thing _is_|Define what a thing _can do_|
|**Stands alone?**|Yes|No|
|**Has `__init__`?**|Usually|Rarely|
|**Relationship**|`is-a`|`can-do`|

## Real-World Feel — Django-style View Mixins

```python
class LoginRequiredMixin:
    def check_auth(self, user):
        if not user.get("is_logged_in"):
            raise PermissionError("Login required")

class PermissionMixin:
    def check_permission(self, user, role):
        if user.get("role") != role:
            raise PermissionError(f"Must be {role}")

class View:
    def get(self, user):
        print("Rendering page...")

# Compose exactly what you need
class AdminView(LoginRequiredMixin, PermissionMixin, View):
    def get(self, user):
        self.check_auth(user)
        self.check_permission(user, "admin")
        super().get(user)
```

```python
admin = {"is_logged_in": True, "role": "admin"}
AdminView().get(admin)   # Rendering page...

guest = {"is_logged_in": False}
AdminView().get(guest)   # PermissionError: Login required
```
#### Naming Convention
By convention, mixins are suffixed with **`Mixin`** so intent is clear:

```python
class TimestampMixin: ...
class CacheMixin: ...
class ValidationMixin: ...
class ReprMixin: ...
```
#### Key Takeaways
- Mixins are **reusable behaviour chunks** — write once, attach anywhere
- They follow **SRP** — each mixin does one thing
- They prevent **copy-pasting** the same methods across unrelated classes
- Always place mixins **to the left** of the base class: `class Foo(Mixin, Base)`
---
# 3. **Liskov Substitution Principle (LSP)**

simply put -> A behavioral notion of subtyping" (1994). 
A core tenet of the paper is that "a subtype (must) preserve the behaviour of the supertype methods and also all invariant and history properties of its supertype". 
In essence, a function accepting a supertype should also accept all its subtypes with no modification.

Any subclass should be a **drop-in replacement** for its parent. Code that works with the parent must work with the child — **without surprises**.

*  Your Example — The Problem
```python
class View:
    def get(self, request) -> Response:          # 1 argument
        ...
class TemplateView(View):
    def get(self, request, template_file: str) -> Response:  # 2 arguments 
        ...
def render(view: View, request) -> Response:
    return view.get(request)   # breaks when view is a TemplateView!
```

```python
render(View(), request)          # ✅ works
render(TemplateView(), request)  # ❌ TypeError — missing template_file
```

`TemplateView` **claims** to be a `View` but can't be used as one.

#### The Fix — Keep the Signature Contract

Move `template_file` to `__init__` so `.get()` stays compatible:

```python
from dataclasses import dataclass

@dataclass
class Response:
    status: int
    content_type: str
    body: str


class View:
    content_type = "text/plain"
    def render_body(self) -> str:
        return "Welcome to my web site"
    def get(self, request) -> Response:
        return Response(
            status=200,
            content_type=self.content_type,
            body=self.render_body()
        )


class TemplateView(View):
    content_type = "text/html"

    def __init__(self, template_file: str):   # ✅ extra data goes here
        self.template_file = template_file

    def get(self, request) -> Response:       # ✅ same signature as parent
        with open(self.template_file) as fd:
            return Response(
                status=200,
                content_type=self.content_type,
                body=fd.read()
            )


def render(view: View, request) -> Response:
    return view.get(request)   # works for ALL View subtypes ✅
```

```python
render(View(), request)                        # ✅ works
render(TemplateView("index.html"), request)    # ✅ works
```
##### Another Classic LSP Violation — The Rectangle/Square Problem

```python
#  Violates LSP
class Rectangle:
    def __init__(self, width: float, height: float):
        self.width = width
        self.height = height

    def area(self) -> float:
        return self.width * self.height

class Square(Rectangle):
    def __init__(self, side: float):
        super().__init__(side, side)

    def set_width(self, width: float):   #  breaks Square's constraint
        self.width = width               #    now width != height

def scale_width(rect: Rectangle, factor: float):
    rect.set_width(rect.width * factor)
    return rect.area()

# Expected: 4 * 2 = 8 ... but Square is now 8x4 = 32? No — inconsistent!
scale_width(Square(4), 2)   # behaviour is surprising and wrong
```

#### Fix — Don't force the hierarchy
```python
from abc import ABC, abstractmethod

class Shape(ABC):
    @abstractmethod
    def area(self) -> float:
        pass

class Rectangle(Shape):
    def __init__(self, width: float, height: float):
        self.width = width
        self.height = height

    def area(self) -> float:
        return self.width * self.height

class Square(Shape):           # Square is a Shape, NOT a Rectangle
    def __init__(self, side: float):
        self.side = side

    def area(self) -> float:
        return self.side ** 2
```
##### LSP Violation Checklist

|Smell|What it means|
|---|---|
|Subclass **adds required parameters** to a method|Breaks callers expecting parent's signature|
|Subclass **raises new exceptions** parent doesn't|Callers aren't prepared to handle them|
|Subclass **ignores or weakens** a method|e.g. overrides `save()` with `pass`|
|Subclass **changes return type** incompatibly|Callers expect parent's return type|
|You need `isinstance()` checks in calling code|Subtypes aren't truly substitutable|
#### takeaways 
- LSP is about **behavioural compatibility**, not just type compatibility
- A subclass can **extend** behaviour but must **honour** the parent's contract
- The fix is usually: move extra data to `__init__`, not into method signatures
- `mypy` / type checkers are your first line of defence — they catch signature mismatches before runtime
- If a subclass can't fully substitute its parent, **reconsider the hierarchy** (like Square/Rectangle)
---

# 4. Interface Segregation Principle (ISP) in Python

> **"Keep interfaces small so that users don't end up depending on things they don't need."** — Robert C. Martin (Uncle Bob)

Don't force a class to implement methods it doesn't use. **Split large interfaces into small, focused ones** and let classes pick only what they need.
#### Your Example  The Problem

```python
import abc

class Persistable(metaclass=abc.ABCMeta):
    @property
    @abc.abstractmethod
    def data(self) -> bytes: ...

    @classmethod
    @abc.abstractmethod
    def load(cls, name: str): ...

    @abc.abstractmethod
    def save(self) -> None: ...      # ❌ PDFDocument doesn't need this!
```

```python
class PDFDocument(Persistable):
    @property
    def data(self) -> bytes: ...

    @classmethod
    def load(cls, name: str): ...

    # save() not implemented...
```

```
TypeError: Can't instantiate abstract class PDFDocument with abstract method save
```

`PDFDocument` is **forced** to implement `save()` even though it only serves files — it never writes them.

---
####  The Fix — Segregate into Smaller Interfaces

```python
import abc

class DataCarrier(metaclass=abc.ABCMeta):
    """Carries a raw data payload"""
    @property
    def data(self) -> bytes: ...


class Loadable(DataCarrier):
    """Can load data from storage"""
    @classmethod
    @abc.abstractmethod
    def load(cls, name: str): ...


class Saveable(DataCarrier):
    """Can save data to storage"""
    @abc.abstractmethod
    def save(self) -> None: ...

class PDFDocument(Loadable):
    @property
    def data(self) -> bytes:
        ...  # read bytes

    @classmethod
    def load(cls, name: str):
        ...  # load from disk


class UploadedDocument(Loadable, Saveable):
    @property
    def data(self) -> bytes: ...

    @classmethod
    def load(cls, name: str): ...

    def save(self) -> None: ...
```

##### Another Simple Example — Worker Roles
#### Violating ISP

```python
class Worker(abc.ABCMeta):
    @abstractmethod
    def work(self): ...

    @abstractmethod
    def eat(self): ...       # Robots don't eat!

    @abstractmethod
    def sleep(self): ...     # Robots don't sleep!


class Robot(Worker):
    def work(self): print("Robot working...")
    def eat(self): pass      # ❌ dummy method — useless
    def sleep(self): pass    # ❌ dummy method — useless
```
##### Following ISP

```python
from abc import ABC, abstractmethod

class Workable(ABC):
    @abstractmethod
    def work(self): ...

class Restable(ABC):
    @abstractmethod
    def eat(self): ...
    @abstractmethod
    def sleep(self): ...
class Human(Workable, Restable):    # Humans do both ✅
    def work(self):  print("Human working...")
    def eat(self):   print("Human eating...")
    def sleep(self): print("Human sleeping...")
class Robot(Workable):              # Robots only work ✅
    def work(self): print("Robot working...")
```

---
##### ISP vs SRP — What's the Difference?

||SRP|ISP|
|---|---|---|
|**Applies to**|Classes (implementations)|Interfaces (abstractions)|
|**Question asked**|Does this class do one thing?|Does this interface ask for only what's needed?|
|**Symptom of violation**|Class has unrelated methods|Class forced to implement unused methods|
|**Fix**|Split the class|Split the interface|

They are complementary — **SRP cleans up classes, ISP cleans up their contracts**.

---

| Signal                                            | What it suggests                         |
| ------------------------------------------------- | ---------------------------------------- |
| `pass` or `raise NotImplementedError` in a method | Forced to implement something not needed |
| "We don't need this _yet_" when designing an ABC  | ISP violation in the making              |
| One abstract class with 5+ abstract methods       | Likely needs to be split                 |
| Subclasses implement totally different subsets    | Interface is too broad                   |

---
#### Key Takeaways

- Python uses **Abstract Base Classes** where other languages use interfaces — same ISP principle applies
- A large ABC that forces irrelevant `pass` implementations is a **design smell**
- Small, composable ABCs let classes **opt in** to only what they need
- When requirements grow (e.g. adding upload), just **add the mixin** — existing classes are untouched

--- 

# 5. Dependency Inversion Principle (DIP) in Python

> **"Depend upon abstractions, not concrete details."** — Uncle Bob

#### The Core Idea

High-level code shouldn't be locked to specific low-level implementations. Both should talk through an **abstraction** (a shared interface/contract).

---

## Your Example — Key Insight

`csv.writer` doesn't care _what_ object you pass it — it only depends on the abstraction: **"does it have a `.write()` method?"**

```python
class Echo:
    def write(self, value):
        return value          # doesn't store — just returns the row instantly
```

```python
writer = csv.writer(Echo())   # ✅ csv.writer depends on .write() — not StringIO specifically
```

You exploited the fact that `csv.writer` is already DIP-compliant — swap `StringIO` for `Echo` and it just works.

---
## Cleaner Example

### ❌ Violating DIP — hardcoded to a concrete class

```python
class MySQLDatabase:
    def save(self, data: str):
        print(f"Saving '{data}' to MySQL")

class UserService:
    def __init__(self):
        self.db = MySQLDatabase()    # ❌ tightly coupled to MySQL

    def create_user(self, name: str):
        self.db.save(name)
```

Switching to Postgres means **modifying** `UserService`.

---

### ✅ Following DIP — depend on an abstraction

```python
from abc import ABC, abstractmethod

class Database(ABC):               # the abstraction
    @abstractmethod
    def save(self, data: str): ...

class MySQLDatabase(Database):
    def save(self, data: str):
        print(f"Saving '{data}' to MySQL")

class PostgresDatabase(Database):
    def save(self, data: str):
        print(f"Saving '{data}' to Postgres")

class UserService:
    def __init__(self, db: Database):  
        self.db = db

    def create_user(self, name: str):
        self.db.save(name)
```

```python
UserService(MySQLDatabase()).create_user("Alice") 
UserService(PostgresDatabase()).create_user("Alice")
```

##### Key Takeaways
||❌ Bad|✅ Good|
|---|---|---|
|Depends on|Concrete class|Abstraction (ABC / Protocol)|
|Swapping implementation|Requires code change|Just inject a different object|
|Testability|Hard (real DB in tests)|Easy (inject a mock)|

- The `Echo` trick works **because** `csv.writer` already follows DIP — it asks for `.write()`, not `StringIO`
- Inject dependencies via `__init__` — this is called **Dependency Injection**, the practical tool for achieving DIP

---
# 6. Dont reapet yourself [DRY]

> **"Every piece of knowledge must have a single, unambiguous representation in a system. Single Source of Truth."** — Andrew Hunt & David Thomas

---
##### The Core Idea
Duplicate code means **multiple places to update** when logic changes. The fix is a good abstraction that handles the variations in one place.

---
### Your Example
##### Violating DRY

```python
@dataclass
class Developer:
    def __init__(self, experience: float, github_link: str): ...

@dataclass
class Manager:
    def __init__(self, experience: float, github_link: str): ...  # identical ❌

def get_developer_list(developers: List[Developer]) -> List[Dict]: ...
def get_manager_list(managers: List[Manager]) -> List[Dict]: ...   # identical logic ❌
``` 


 **Two classes, two functions — all doing the same thing. Change the data shape? Update **four** places.**
###  Following DRY

```python
@dataclass
class Employee:
    def __init__(self, experience: float, github_link: str):
        self._experience = experience
        self._github_link = github_link

    @property
    def experience(self) -> float:
        return self._experience

    @property
    def github_link(self) -> str:
        return self._github_link


def get_employee_list(employees: List[Employee]) -> List[Dict]:
    return [{'experience': e.experience, 'github_link': e.github_link}
            for e in employees]
```

```python
company_developers = [Employee(2.5, 'https://github.com/1'), ...]
company_managers   = [Employee(4.5, 'https://github.com/3'), ...]

get_employee_list(company_developers)   # ✅
get_employee_list(company_managers)     # ✅ same function
```

One class, one function — change the logic **once**, it applies everywhere.
## Another Common DRY Violation — Repeated Logic

### ❌

```python
def get_senior_developers(employees):
    return [e for e in employees if e.experience > 5 and e.role == "developer"]

def get_senior_managers(employees):
    return [e for e in employees if e.experience > 5 and e.role == "manager"]
```

### ✅

```python
def get_senior_employees(employees, role: str):
    return [e for e in employees if e.experience > 5 and e.role == role]
```

---

## The Important Caveat

> **Bad abstractions can be worse than duplicate code.**

Don't force unrelated things together just to avoid repetition. Ask:

- Do these things share the **same concept** or just look similar by coincidence?
- If one changes, should the other change too?

If yes → abstract. If no → keep them separate.

---

## Key Takeaways

||❌ Wet (Write Everything Twice)|✅ DRY|
|---|---|---|
|Change needed|Update N places|Update 1 place|
|Risk of bugs|High — easy to miss a copy|Low|
|Code size|Bloated|Lean|

- DRY applies to **logic, data, and configuration** — not just classes
- The goal isn't fewer lines — it's **one source of truth** per concept
- When you find yourself copy-pasting, stop and ask: _"what's the abstraction here?"_






# What is following: 

### 1. in Creational Pattern

![[Pasted image 20260306120226.png]]

### 2. Structural Pattens
![[Pasted image 20260306120533.png]]

### 3. Behavorial Patterns
![[Pasted image 20260306120749.png]]



## **Class based relationship model**

1. **Association** 
	- Association represents a relationship between two classes where **one object uses, communicates with, or references another**.  “One object need to know about the existence of another object to perform its responsibilities”. 
	-  Association reflects a **"has-a"** or **"uses-a"** relationship.
	- Associated objects are **loosely coupled** and can exist **independently** of one another.
	- The association can be **unidirectional** or **bidirectional**, and can follow different **multiplicity** patterns (1-to-1, 1-to-many, etc.).
	
2. **Composition** [Structural] 
	- **Composition** is a special type of association that signifies **strong ownership** between objects. The “whole” class is **fully responsible** for creating, managing, and destroying the “part” objects. In fact, the parts **cannot exist without** the whole. 
	-  Represents a **strong “has-a”** relationship.
	- The **whole owns** the part and **controls its lifecycle**.
	- When the whole is destroyed, the **parts are also destroyed**.
	- The parts are **not shared** with any other object.
	- The part has **no independent meaning or identity** outside the whole

3. **Aggregation** 
	- Aggregation is a specialized form of association that models a **whole-part relationship** with **loose ownership**. One class (the "whole") contains references to other class objects (the "parts"), but the parts can exist independently of the whole
	-  The **whole** and the **part** are logically connected through a container-contained hierarchy.
	- The **part can exist independently** of the whole.
	- The **whole does not create or destroy** the part.
	- The **part can be shared** among multiple wholes.
	- Both the whole and the part can be **created and destroyed independently**.

4. **Dependency** : 
	- exists when **one class relies on another** to fulfill a responsibility, but does so **without retaining a permanent reference** to it.
	- **Short-lived**: The relationship exists **only during method execution**.
	- **No ownership**: The dependent class does **not store** the other as a field.
	- **"Uses-a" relationship**: The class uses another to **accomplish a task**, but does not retain it.