Начало чтения книги Intro to Python for Computer Science and Data Science(https://drive.google.com/file/d/1rXkYFjw1iKbXCra_B4Ykm0AMRgo6v93w/view?fbclid=IwAR2lg9omGaAsG3g1ZhHQHja8_uxkZ7QddnOUSxfoceRXShU1V_bl4V63xCQ) 
https://github.com/Moataz-Elmesmary/Data-Science-Roadmap
Начало использования [[Jupyter Notebook]], [[Marimo]]
Изучение библиотек в [[Python]], таких как [[Pandas]], [[NumPy]]

Для закрепления изученного LeetCode
Kaggle для Pandas


## **Цикл while в Python**
```python
x = 1 
while x < 5:     
	print(x)     
	x += 1  
```
## **Цикл for в Python**
```python
num_list = [14, 101, -7, 0]
for number in num_list:
    print(number)
    
for i in range(3):
    print(i)

num_list = [i for i in range(1, 11)]
print(num_list)

string = 'Hi loop!'
for i in string:
    if i == ',':
        break
    print(i)

for i in range(1, 10):
    if i%2 == 0 or i%3 == 0:
        continue
    print(i)
```
## List Comprehension
```python
squares = [n**2 for n in range(10)]
print(squares)
#[0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
```

## Синтаксис функций
```python
def имя_функции (аргументы):
    тело_функции
    return результат
```
У функции имеется область видимости:
### **Локальная область (local scope)**
```python
def sum(a, b):
    c = a + b
    return c
```
### **Область объемлющей функции (enclosing function scope)**
```python
def make_counter():
    # Объявляем переменную count в объемлющей функции
    count = 0

    def counter():
        # Указываем, что count находится в объемлющей функции
        nonlocal count 
        count += 1
        return count

    return counter

# Создаём счётчик
call_counter = make_counter()
```
Функция имеет вложенную функцию.
### **Глобальная область (global scope)**
```python
cake_count = 10

def modify_cake():
    global cake_count
    # Изменяем значение глобальной переменной
    cake_count = 15

modify_cake()
```
Используется глобальная переменная.
### **Аргументы переменной длины (*args и **kwargs)**
- *args используют, когда неясно, сколько позиционных аргументов у нас есть. Звёздочка * перед args указывает на то, что все позиционные аргументы, переданные при вызове функции, должны быть собраны в кортеж tuple и присвоены args.
- **kwargs используют, чтобы передать именованные аргументы в виде словаря (dictionary), когда мы не знаем, сколько их у нас. Две звёздочки (**) перед kwargs означают, что все именованные аргументы должны быть собраны в словарь и присвоены kwargs.
```python
def greet(greeting, *args, **kwargs):
    for name in args:
        message = f'{greeting}, {name}!'
        if 'mood' in kwargs:
            message += f 'Ты чувствуешь себя {kwargs['mood']}.'
        print(message)

# Пример использования
greet('Привет', 'Катя', 'Лена', 'Вика', mood= 'весело')
greet('Здравствуйте', 'Саша, 'Таня')
```
## **Lambda-функции**
Лямбда-функции (lambda-функции) ― это безымянные функции, которые могут быть определены в одной строке кода. Выше мы упомянули, что название функции используют, чтобы вызвать функцию повторно. Лямбда-функцию нельзя переиспользовать ― у неё нет имени, по которому её можно вызвать. Обычно их используют там, где требуется передать небольшую функцию в качестве аргумента.
```python
is_even = lambda num: num % 2 == 0

print(is_even(4))  # Вывод: True
print(is_even(7))  # Вывод: False
```

## Функция map() в Python
Это встроенный инструмент для преобразования последовательностей данных: #списки , кортежи, словари, множества и других итерируемых объектов.
```python
# Создадим два списка с числами
numbers1 = [1, 2, 3]
numbers2 = [10, 20, 30]

# Напишем функцию для сложения двух чисел
def add_numbers(x, y):
    return x + y

# Правильное использование map() с двумя списками ✅
result = list(map(add_numbers, numbers1, numbers2))
print(result)  # [11, 22, 33]

#С использованием lambda-функций
numbers = [1, 2, 3, 4]
cubic = map(lambda x: x ** 3, numbers)
print(list(cubic))  # [1, 8, 27, 64]
```



## Работа с виртуальным окружением venv.
Установка сторонних пакетов глобально в Python это плохая идея, поэтому используем  виртуальное окружение.
```Bash
python -m venv virtualenv
```
virtualenv - название виртуального окружения.
Для его активации:
```Bash 
cd venv\Scripts\activate.bat
```
Виртуальное окружение активировано и теперь можно устанавливать в него пакеты используя pip:
```Bash
pip install Pandas
```
## Строки
Для изменения регистра:
```python
str='hello'
str.upper()
str.lower()
```
Поиск индекса первого вхождения:
```python
str='he is my friend'
substr='my'
str.index(substr)
```
Разбиение строки на мелкие(токены):
```python
str='Better call Saul'
str.split() #'Better','call','Saul'
```
## Словари
```python
numbers = {'one':1, 'two':2, 'three':3}

planets = ['Mercury', 'Venus', 'Earth', 'Mars', 'Jupiter', 'Saturn', 'Uranus', 'Neptune']
planet_to_initial = {planet: planet[0] for planet in planets}
```
