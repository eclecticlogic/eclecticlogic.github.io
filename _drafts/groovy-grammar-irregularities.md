
if you have a map of string, list<int> you can't do 
then a['b']?[0] to get value from list if value is non-null for key.

---

Operator precedence produces odd results:

println 3 * 4 + 5

list << some-boolean condition ? 'a' : 'b'

---


You can't use blocks {} since they conflict with closure syntax.


---

You can create a map like ['a': 25] but you can't create a map as [stringReturningFunction(): 45]



