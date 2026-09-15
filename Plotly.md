Marimo Pyplot notebook
Библиотека для интерактивной визуализации.
Установка:
```Bash
pip install plotly
```
Импорт библиотек:
```python
import plotly.graph_objs as go  
import plotly.express as px  
from plotly.subplots import make_subplots
```
## Линейный график
Для примера график y=x^2
```python
import plotly
import plotly.graph_objs as go
import plotly.express as px
from plotly.subplots import make_subplots
import numpy as np
import pandas as pd

x = np.arange(0, 5, 0.1)
def f(x):
    return x**2

px.scatter(x=x, y=f(x)).show()
```
![[Pasted image 20260327220219.png]]
## Создание фигуры и нанесение на нее объектов:
```python 
fig = go.Figure()
fig.add_trace(go.Scatter(x=x,y=f(x)))
fig.add_trace(go.Scatter(x=x,y=2*x))
```
![[Pasted image 20260329131046.png]]
Функция `add_trace`  добавляет на график то, что захочешь, также позволяет добавлять сразу 2 графика. Также появилась легенда(можно убрать графики).
```python
fig.add_trace(go.Scatter(x=x,y=f(x),name='f(x)==x**2'))
```
![[Pasted image 20260329131636.png|325]]
Графики поддерживают подпись.
```python
fig.update_layout(margin=dict(l=0, r=0, t=0, b=0),title="Plot Title",
xaxis_title="x Axis Title",
yaxis_title="y Axis Title") #для того, чтобы убрать отступы у графика и добавить подпись к осям и самому графику
```
## Оси:
Создание осей коордиант и выделение их цветом, также создание интервала:
```python
fig.update_yaxes(range=[-1,5],zeroline=True, zerolinewidth=2, zerolinecolor='LightPink')
fig.update_xaxes(range=[0,5],zeroline=True, zerolinewidth=2, zerolinecolor='Red')
```
![[Pasted image 20260329134217.png]]
## Фигура с несколькими графиками:
```python
fig1 = make_subplots(rows = 1, cols = 2)
fig1.update_yaxes(range=[-1,5],zeroline=True, zerolinewidth=2, zerolinecolor='LightGreen',row=1,col=2) # col для указания на каком графике будет
fig1.update_xaxes(range=[-1,10],zeroline=True, zerolinewidth=2, zerolinecolor='Purple')
fig1.add_trace(go.Scatter(x=x,y=np.sin(x)),1,1)
fig1.add_trace(go.Scatter(x=x,y=np.cos(x)),1,2)
fig1.update_layout(margin=dict(l=0, r=0, t=0, b=0),title="Plot Title",
xaxis_title="x Axis Title",
yaxis_title="y Axis Title")
```
![[Pasted image 20260329135456.png]]
Функция `make_subplots` позволяет создавать на одной фигуре сразу несколько графиков, далее при использовании `add_trace` в конце добавляем 2 агрумента, которые отвечают за то, где будет нарисован график.
## Тепловая карта
```python
fig2 = go.Figure()
def h(x):
    return np.sin(x)
    
fig2.add_trace(go.Scatter(x=x, y=f(x), mode='lines+markers',  name='f(x)=x<sup>2</sup>', 
                         marker=dict(color=h(x), colorbar=dict(title="h(x)=sin(x)"),colorscale = 'Inferno')
                        ))

fig2.update_layout(legend_orientation="h",
                  legend=dict(x=.5, xanchor="center"),
                  margin=dict(l=0, r=0, t=0, b=0))
fig2.update_traces(hoverinfo="all", hovertemplate="Аргумент: %{x}<br>Функция: %{y}")
fig2.show()
```
![[Pasted image 20260331211519.png]]
## Анимация
