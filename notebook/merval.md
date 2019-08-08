
# Merval


```python
import pandas as pd
import numpy as np

# Cargamos los datos del índice desde el archivo .csv
data = pd.read_csv("merval.csv")

# Veamos los últimos datos
data.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Fecha</th>
      <th>Último</th>
      <th>Apertura</th>
      <th>Máximo</th>
      <th>Mínimo</th>
      <th>Vol.</th>
      <th>% var.</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>02.08.2019</td>
      <td>41.359,15</td>
      <td>41.410,98</td>
      <td>41.651,40</td>
      <td>40.572,68</td>
      <td>-</td>
      <td>-0,13%</td>
    </tr>
    <tr>
      <th>1</th>
      <td>01.08.2019</td>
      <td>41.410,98</td>
      <td>42.057,77</td>
      <td>42.492,78</td>
      <td>41.220,28</td>
      <td>-</td>
      <td>-1,54%</td>
    </tr>
    <tr>
      <th>2</th>
      <td>31.07.2019</td>
      <td>42.057,77</td>
      <td>42.463,24</td>
      <td>42.722,32</td>
      <td>41.804,62</td>
      <td>-</td>
      <td>-0,95%</td>
    </tr>
    <tr>
      <th>3</th>
      <td>30.07.2019</td>
      <td>42.463,24</td>
      <td>42.785,46</td>
      <td>42.836,75</td>
      <td>42.417,96</td>
      <td>-</td>
      <td>-0,75%</td>
    </tr>
    <tr>
      <th>4</th>
      <td>29.07.2019</td>
      <td>42.785,46</td>
      <td>41.983,74</td>
      <td>43.069,98</td>
      <td>41.743,16</td>
      <td>-</td>
      <td>1,91%</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Cantidad de muestras (días) que tenemos del índice MERVAL
len(data)
```




    2449




```python
# Necesitamos convertir cada variación a float para poder analizar los datos ya que eran strings.
data["% var."] = data["% var."].apply(lambda x: float(x.replace(",",".")[0:-1]))
```


```python
# Días que el Merval se mantuvo estacionario
# Cantidad de estados (S)
data[data["% var."] == 0]["% var."].count()
```




    22




```python
# Días que el Merval subió
# Cantidad de estados (U)
data[data["% var."] > 0]["% var."].count()
```




    1312




```python
# Días que el Merval bajó
# Cantidad de estados (D)
data[data["% var."] < 0]["% var."].count()
```




    1115




```python
# Necesitamos saber la correlación de estados, es decir, en que estado estaba y a cual se movió
# Para poder armar luego la matriz de transición
# Para esto necesitamos saber en que estado estamos y cual fue el anterior
conditions = [
    (data["% var."] > 0),
    (data["% var."] < 0),
    (data["% var."] == 0)
]
choices = ["U", "D", "S"]
data["Actual"] = np.select(conditions, choices, default="black")
```


```python
# Ya tenemos una columna con el estado actual
data.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Fecha</th>
      <th>Último</th>
      <th>Apertura</th>
      <th>Máximo</th>
      <th>Mínimo</th>
      <th>Vol.</th>
      <th>% var.</th>
      <th>Actual</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>02.08.2019</td>
      <td>41.359,15</td>
      <td>41.410,98</td>
      <td>41.651,40</td>
      <td>40.572,68</td>
      <td>-</td>
      <td>-0.13</td>
      <td>D</td>
    </tr>
    <tr>
      <th>1</th>
      <td>01.08.2019</td>
      <td>41.410,98</td>
      <td>42.057,77</td>
      <td>42.492,78</td>
      <td>41.220,28</td>
      <td>-</td>
      <td>-1.54</td>
      <td>D</td>
    </tr>
    <tr>
      <th>2</th>
      <td>31.07.2019</td>
      <td>42.057,77</td>
      <td>42.463,24</td>
      <td>42.722,32</td>
      <td>41.804,62</td>
      <td>-</td>
      <td>-0.95</td>
      <td>D</td>
    </tr>
    <tr>
      <th>3</th>
      <td>30.07.2019</td>
      <td>42.463,24</td>
      <td>42.785,46</td>
      <td>42.836,75</td>
      <td>42.417,96</td>
      <td>-</td>
      <td>-0.75</td>
      <td>D</td>
    </tr>
    <tr>
      <th>4</th>
      <td>29.07.2019</td>
      <td>42.785,46</td>
      <td>41.983,74</td>
      <td>43.069,98</td>
      <td>41.743,16</td>
      <td>-</td>
      <td>1.91</td>
      <td>U</td>
    </tr>
  </tbody>
</table>
</div>




```python
for i in range(1, len(data)):
    data.loc[i-1, "Anterior"] = data.loc[i, "Actual"]
```


```python
# Ahora ya tenemos en cada estado cual fue el anterior, ya tenemos todos los datos para calcular nuestra matriz
data.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Fecha</th>
      <th>Último</th>
      <th>Apertura</th>
      <th>Máximo</th>
      <th>Mínimo</th>
      <th>Vol.</th>
      <th>% var.</th>
      <th>Actual</th>
      <th>Anterior</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>02.08.2019</td>
      <td>41.359,15</td>
      <td>41.410,98</td>
      <td>41.651,40</td>
      <td>40.572,68</td>
      <td>-</td>
      <td>-0.13</td>
      <td>D</td>
      <td>D</td>
    </tr>
    <tr>
      <th>1</th>
      <td>01.08.2019</td>
      <td>41.410,98</td>
      <td>42.057,77</td>
      <td>42.492,78</td>
      <td>41.220,28</td>
      <td>-</td>
      <td>-1.54</td>
      <td>D</td>
      <td>D</td>
    </tr>
    <tr>
      <th>2</th>
      <td>31.07.2019</td>
      <td>42.057,77</td>
      <td>42.463,24</td>
      <td>42.722,32</td>
      <td>41.804,62</td>
      <td>-</td>
      <td>-0.95</td>
      <td>D</td>
      <td>D</td>
    </tr>
    <tr>
      <th>3</th>
      <td>30.07.2019</td>
      <td>42.463,24</td>
      <td>42.785,46</td>
      <td>42.836,75</td>
      <td>42.417,96</td>
      <td>-</td>
      <td>-0.75</td>
      <td>D</td>
      <td>U</td>
    </tr>
    <tr>
      <th>4</th>
      <td>29.07.2019</td>
      <td>42.785,46</td>
      <td>41.983,74</td>
      <td>43.069,98</td>
      <td>41.743,16</td>
      <td>-</td>
      <td>1.91</td>
      <td>U</td>
      <td>U</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Procedemos a obtener los datos para armar la matriz
# Cantidad de transiciones del estado U al estado U.

data[(data["Anterior"] == "U") & (data["Actual"] == "U")].count()
```




    Fecha       736
    Último      736
    Apertura    736
    Máximo      736
    Mínimo      736
    Vol.        736
    % var.      736
    Actual      736
    Anterior    736
    dtype: int64




```python
# Cantidad de transiciones del estado U al estado S.
data[(data["Anterior"] == "U") & (data["Actual"] == "S")].count()
```




    Fecha       15
    Último      15
    Apertura    15
    Máximo      15
    Mínimo      15
    Vol.        15
    % var.      15
    Actual      15
    Anterior    15
    dtype: int64




```python
# Cantidad de transiciones del estado U al estado D.
data[(data["Anterior"] == "U") & (data["Actual"] == "D")].count()
```




    Fecha       561
    Último      561
    Apertura    561
    Máximo      561
    Mínimo      561
    Vol.        561
    % var.      561
    Actual      561
    Anterior    561
    dtype: int64




```python
# Cantidad de transiciones del estado S al estado U.
data[(data["Anterior"] == "S") & (data["Actual"] == "U")].count()
```




    Fecha       11
    Último      11
    Apertura    11
    Máximo      11
    Mínimo      11
    Vol.        11
    % var.      11
    Actual      11
    Anterior    11
    dtype: int64




```python
# Cantidad de transiciones del estado S al estado S.
data[(data["Anterior"] == "S") & (data["Actual"] == "S")].count()
```




    Fecha       1
    Último      1
    Apertura    1
    Máximo      1
    Mínimo      1
    Vol.        1
    % var.      1
    Actual      1
    Anterior    1
    dtype: int64




```python
# Cantidad de transiciones del estado S al estado D.
data[(data["Anterior"] == "S") & (data["Actual"] == "D")].count()
```




    Fecha       10
    Último      10
    Apertura    10
    Máximo      10
    Mínimo      10
    Vol.        10
    % var.      10
    Actual      10
    Anterior    10
    dtype: int64




```python
# Cantidad de transiciones del estado D al estado U.
data[(data["Anterior"] == "D") & (data["Actual"] == "U")].count()
```




    Fecha       565
    Último      565
    Apertura    565
    Máximo      565
    Mínimo      565
    Vol.        565
    % var.      565
    Actual      565
    Anterior    565
    dtype: int64




```python
# Cantidad de transiciones del estado D al estado S.
data[(data["Anterior"] == "D") & (data["Actual"] == "S")].count()
```




    Fecha       6
    Último      6
    Apertura    6
    Máximo      6
    Mínimo      6
    Vol.        6
    % var.      6
    Actual      6
    Anterior    6
    dtype: int64




```python
# Cantidad de transiciones del estado D al estado D.
data[(data["Anterior"] == "D") & (data["Actual"] == "D")].count()
```




    Fecha       543
    Último      543
    Apertura    543
    Máximo      543
    Mínimo      543
    Vol.        543
    % var.      543
    Actual      543
    Anterior    543
    dtype: int64




```python
# Ya podemos armar la matriz de cantidad de estados

matriz_estados = np.matrix([[736,15,561], [11, 1, 10], [565, 6, 543]])
matriz_estados
```




    matrix([[736,  15, 561],
            [ 11,   1,  10],
            [565,   6, 543]])




```python
# Calculamos las probabilidades y obtenemos la matriz de transición

matriz_transiciones = matriz_estados * 1/matriz_estados.sum(1)
matriz_transiciones
```




    matrix([[0.56097561, 0.01143293, 0.42759146],
            [0.5       , 0.04545455, 0.45454545],
            [0.50718133, 0.005386  , 0.48743268]])




```python
# Para obtener las probabilidades de los estados inciales
# dividimos la suma de las ocurrencias de cada estados por la cantidad total de estados

p_U_0 = (736 + 15 + 561) / 2449
p_S_0 = (11 + 1 + 10) / 2449
p_D_0 = (565 + 6 + 543) / 2449
```


```python
estado_inicial = np.matrix([p_U_0, p_S_0, p_D_0])
```


```python
estado_inicial
```




    matrix([[0.53572887, 0.00898326, 0.45487954]])




```python
# Si comenzamos en el estado U las probabilidades al cabo de 5 dias quedan

np.matrix([1, 0, 0]) * matriz_transiciones ** 5
```




    matrix([[0.5359479 , 0.008987  , 0.45506511]])




```python
# Si comenzamos en el estado S las probabilidades al cabo de 5 dias quedan

np.matrix([0, 1, 0]) * matriz_transiciones ** 5
```




    matrix([[0.53594733, 0.00898691, 0.45506576]])




```python
# Si comenzamos en el estado D las probabilidades al cabo de 5 dias quedan

np.matrix([0, 0, 1]) * matriz_transiciones ** 5
```




    matrix([[0.5359475 , 0.00898685, 0.45506565]])




```python
# Para saber el estado asintotico multiplicamos la matriz de transicion reiteradas veces

matriz_transiciones ** 999
```




    matrix([[0.53594771, 0.00898693, 0.45506536],
            [0.53594771, 0.00898693, 0.45506536],
            [0.53594771, 0.00898693, 0.45506536]])




```python
# Cantidad de días dentro de 5 dias que esperariamos que el indice suba
5 * 0.53594771
```




    2.6797385499999997




```python
# Cantidad de días dentro de 5 dias que esperariamos que el indice se mantenga estacionario
5 * 0.00898693
```




    0.04493465000000001




```python
# Cantidad de días dentro de 5 dias que esperariamos que el indice baje
5 * 0.45506536
```




    2.2753267999999998




```python
# Tiempo que esperaríamos que tarde en salir de una suba hasta que vuelva a subir el indice
1 / 0.56097561
```




    1.7826086948771267




```python
# Tiempo que esperaríamos que tarde en salir de un estado estacionario hasta que vuelva a estar estacionario
1 / 0.04545455
```




    21.99999780000022




```python
# Tiempo que esperaríamos que tarde en salir de una baja hasta que vuelva a bajar el indice
1 / 0.48743268
```




    2.0515653566765364




```python

```
