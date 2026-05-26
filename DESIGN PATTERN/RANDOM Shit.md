Good quality code :-> [zedr/clean-code-python: :bathtub: Clean Code concepts adapted for Python](https://github.com/zedr/clean-code-python?tab=readme-ov-file#introduction) 

Abstract classes in python :

1. child classes do not immediatly needs to implement all the parents methods. they can just pass in the child method too. 
2. @staticmethod - decorator -> allows to create a method that can be used withod creating a class . 

		1. a  = Vehical()
		2. a.honk(123)
		3. --- you just (if honk is @staticmethod)
		4. a = Vehical.honk(123)
3. @abstractclass declare a ABC Class.  Abstract interface class. For the implementation. 
4. 