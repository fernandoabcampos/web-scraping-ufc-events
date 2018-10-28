
<div style="width: 100%; clear: both;">
<div style="float: left; width: 50%;">
<img src="http://www.uoc.edu/portal/_resources/common/imatges/marca_UOC/UOC_Masterbrand.jpg", align="left">
</div>
<div style="float: right; width: 50%;">
<p style="margin: 0; padding-top: 22px; text-align:right;">M2.851 - Tipología y ciclo de vida de los datos aula 1 · Práctica 1</p>
<p style="margin: 0; text-align:right;">2018 · Máster universitario en Ciencia de datos (Data science)</p>
<p style="margin: 0; text-align:right;">Prof. Colaboradora: <b>Laia Subirats Maté</b></p>
<p style="margin: 0; text-align:right; padding-button: 100px;">Alumno: <b>Fernando Antonio Barbeiro Campos</b> - <a href="">fbarbeiro@uoc.edu</a></p>
</div>
</div>
<div style="width:100%;">&nbsp;</div>



### Dataset
Historial de eventos del UFC (*Ultimate Fighting Championship*)
### Descripción
Un dataset completo con todos los eventos deportivos de MMA (*mixed martial arts*) más famoso del mundo, los deportistas, resultados, entre otros.

### Imagen identificativa

<img src="ufc-results.jpg" alt="UFC results" style="width: 700px;"/>
<center>Figura 1: Resultados y card de peleas de un evento de UFC.</center> 

### Contexto
El conjunto de datos se trata de todos los eventos deportivos de MMA realizados desde la creación del UFC (primer evento en el **12 de Noviembre de 1993**) hasta el último que ocurrió en el **27 de Octubre de 2018**, o sea, **454 eventos**, bien como el resultado de cada una de las peleas que ocurrieron en cada evento - precisamente **4869 luchas**.

### Contenido
El dataset tiene un contenido bastante sencillo que consiste en:
* [String] *event_name:* Nombre del evento
* [String] *weight_class:* Categoría de peso de la pelea
* [String] *fighter_1:* Nombre del competidor 1
* [String] *action:* Acción que ejecutó el competidor 1 sobre el oponente / resultado
* [String] *fighter_2:* Nombre del oponente
* [Integer] *round:* *Round* en que acabó la lucha
* [String] *time:* Tiempo en el *round* que acabó la lucha
* [String] *method:* Como ha sido el método utilizado para ganar

El intervalo de tiempo fuera mencionado arriba, vale resaltar que en cada evento hay una media de 10.7 luchas. La estrategia utilizada para recoger los datos es el resultado de iteraciones en listas:
* Inicialmente, pillamos la [lista de todos los eventos del UFC](https://en.wikipedia.org/wiki/List_of_UFC_events)
* Hacemos un `for` loop en la lista completa y extraemos (con uso de BeautifulSoup) las URLs individuales de cada evento
* Finalmente también haremos una lectura de tabla para finalmente recolectar los datos.

### Agradecimientos
En análisis se realizó entre **17 y 20 de Octubre** y específicamente el enfoque era encontrar puntos donde pudiera haber ilegalidad en la extracción de información de las fuentes de datos. Para ello, estuve mirando especialmente los ficheros de [robots.txt de Wikipedia](https://en.wikipedia.org/robots.txt), que dicho, es nuestra fuente de información, al cual dejamos el agradecimiento y sugiero donaciones para la iniciativa.
Las URLs de lectura no estaban *Disallowed* en el fichero de Robots, por lo tanto, no habría problemas explorarlas. Aun así, he aplicado técnicas como sleep recomienda **[4] Miller, C. (2017)** para evitar la sobrecarga (*throttling*) de la webpage. 

### Inspiración
Desde mi punto de vista, el conjunto de datos es interesante porque el ámbito de su aplicación es bastante amplio, para comprobar, voy a describir escenarios de su aplicación:

1. Primeramente, una aplicación podría ser para uso en periodismo deportivo - para enseñar datos y patrones del deporte.

2. Un deportista, que compete o no en el evento, podría sacar informaciones de métodos más comúnmente utilizados para encerrar una lucha y, de esa manera, prepararse para evitar que hicieran con él o mismo practicar las técnicas para intentar aplicarlas cuando esté competiendo.

3. La preparación física en un deporte es un punto clave. Entender donde suele pasar la mayor parte de fines de luchas puede también ayudar los profesionales de preparación física a entrenar sus atletas para que en cansancio no les quite la oportunidad de victoria.

4. Podríamos identificar patrones de acuerdo con el peso (categoría) de los atletas, es decir, puede que en una categoría más ligera estén acostumbrados a pelearse más de pie, mientras en otras categorías más pasadas suele ocurrir grappling (lucha de suelo). 

Seguramente hay otras numerosas posibilidades de aplicación.


### Licencia
Haciendo un breve estudio de las licencias presentadas, creo que la que se aplicaría más ampliamente a mi estudio sería **CC BY-NC-SA 4.0**, me explico: la licencia en cuestión permite:
* *Compartir*: copiar y redistribuir el material
* *Adaptar*: transformar y cambiar el material

Sin embargo, no permite el uso comercial del mismo - el que, siendo un trabajo de máster, tiene fines más académicos.

Abajo, una imagen de que trata la licencia elegida:
<img src="license.png" alt="License" style="width: 500px; height: 400px;"/>
<center>Figura 2: CC BY-NC-SA 4.0.</center> 

P.D.: Como el los repositorios de Github no había disponibilidad de utilizar la dicha licencia, por allí he definido el uso de **BSD 3-Clause** que básicamente define que las redistribuciones de generadas con base en el proyecto en cuestión deben ser hechas con notificación a priori. Ademas, garantiza que los nombres de los creadores del proyecto inicial no pueden ser usados para promover productos derivados del proyecto inicial.

### Código
En ese apartado, tendremos el código utilizado para la extracción de los datos y al final para la generación del CSV.


```python
import sys
print(sys.version)
```

    3.7.0 (v3.7.0:1bf9cc5093, Jun 26 2018, 23:26:24) 
    [Clang 6.0 (clang-600.0.57)]



```python
# Not necessary at all, but to demonstrate that I'm aware that BeautifulSoup4 must be installed
!{sys.executable} -m pip install --upgrade pip
!{sys.executable} -m pip install BeautifulSoup4
```

    Requirement already up-to-date: pip in /Library/Frameworks/Python.framework/Versions/3.7/lib/python3.7/site-packages (18.1)
    Requirement already satisfied: BeautifulSoup4 in /Library/Frameworks/Python.framework/Versions/3.7/lib/python3.7/site-packages (4.6.3)



```python
from bs4 import BeautifulSoup
from IPython.core.display import display, HTML
from time import sleep
import requests
import pandas as pd
import os
```


```python
Events = []
base_url = 'https://en.wikipedia.org'
main_url = base_url + '/wiki/List_of_UFC_events'
```


```python
def perform_http_get(url):
    """
    Note:
        This is a simple function that performs an http_get and if the result code is 200 proceed the return.  

    Args:
        url (str): A string.

    Returns:
        BeautifulSoup: a version parsed of the request content
    """
    
    
    r = requests.get(url)
    if r.status_code == 200:
        return BeautifulSoup(r.content, 'html.parser')
        
```


```python
def extract_cell(cells, id_td):
    """
    Note:
        This method is aimed for returning the content of a given cell (td) decoded and without 
        additional blank spaces. 

    Args:
        cells (list): A list representing the table row.
        id_td (int): The index of the wanted cell

    Returns:
        String: the real content of the cell.
    """
    
    return cells[id_td].renderContents().decode().strip()
```


```python
def append_fighter_names(cells_event, info):
    """
    Note:
        This method presents some logic that will be briefly explained here. It turns out that some
        fighters don't have their own wikipedia page, in these cases, there is no link (<a>) within
        their names. Therefore, returning just the content of the cell where this info is present 
        is enough. In other cases, where the link is there, we must extract to update our dictionary.
    Args:
        cells_event (list): A list representing the table row.
        info (dict): The dictionary to be updated
    """
    
    
    
    fighter1 = ''
    fighter2 = ''
    if len(cells_event[1].findAll('a')) == 0:
        fighter1 = extract_cell(cells_event, 1)
    else:
        fighter1 = cells_event[1].find('a').renderContents().decode().strip()

    if len(cells_event[3].findAll('a')) == 0:
        fighter2 = extract_cell(cells_event, 3)
    else:
        fighter2 = cells_event[3].find('a').renderContents().decode().strip()
    
    info.update({"fighter_1" : fighter1})
    info.update({"fighter_2" : fighter2})
```


```python
def extract_row(cells_event, link):
    """
    Note:
        The goal here is to create a dictionary for the given events and also
        to update in our events list (Events.append). We realize that within
        the method we are also invoking the append_fighter_names that will call
        the aforementioned method to treat the fighters' name.

    Args:
        cells_event (list): A list representing the table row.
        link (str): Basically the event name
    """   
    info = {
        "event_name": link.contents[0],
        "weight_class": extract_cell(cells_event, 0),
        "action": extract_cell(cells_event, 2),
        "method": extract_cell(cells_event, 4),
        "round": extract_cell(cells_event, 5),
        "time": extract_cell(cells_event, 6)
    }
    
    append_fighter_names(cells_event, info)
    Events.append(info)
```


```python
def download_event_image(individual_event, link):
    """
    Note:
        The method's name is pretty clear here, however, the objective is to 
        download the images of each event in order to present them at the bottom
        of this current document. In a nutshell, the behaviour expected here
        is to create all the recovered images in a folder called pictures
        which, by the way, we can observe that will be created.

    Args:
        individual_event (BeautifulSoup): A BeautifulSoup representing the page of a single event.
    """  
    
    event_images = individual_event.select('table.infobox a.image img[src]')
    if len(event_images) > 0:
        for img in event_images:
            img_url = 'http:' + img['src']
            r = requests.get(img_url)
            with open('pictures/' + link.contents[0], "wb") as code:
                code.write(r.content)
            
```


```python
def extract_info_individual_event(link):
    """
    Note:
        As you will see, this is the second part of a for loop that I will summarize here:
        First we iterate over the list of UFC events presented in the main web-page (
        https://en.wikipedia.org/wiki/List_of_UFC_events). From the data gathered there
        each individual event will be accessed and we'll collect data from these events.
        
        Needless to say that I'm not iterating over a single web-page, but instead, more than
        450 different web pages, being the first of them the "master one", and this individual
        ones.

    Args:
        link (url): The URL for the individual web-pages of a given list.
    """  
    individual_event = perform_http_get(base_url + link.get('href'))
    table = individual_event.find('table',{'class': 'toccolours'})

    download_event_image(individual_event, link)
   
    if table is not None:
        rows_event = table.findAll('tr')

        for row_event in rows_event:
            cells_event = row_event.findAll('td')
            if len(cells_event) > 0 :
                extract_row(cells_event, link)
                        
```


```python
"""
From my perspective this is a key part of the program and it will trigger 
the execution and the invocation of the previous methods.
Basically we are collecting the "Past Events" of a table in one page and
for each event in this table (currently 454), we will read another 
information as previously explained in different web pages.
By reading another information, I mean: download the event image, read
the card and the results of the event and so on.


Apart from the behaviour expected, in this part of the code, I also tried
to avoid throtling the Wikipedia source by adding some sleeps within each
iteration - it will slow down the process and avoid some common problems.
"""


Events = []
soup = perform_http_get(main_url)
table_past_events = soup.find('table', {'id': 'Past_events'})

rows = table_past_events.findAll('tr')

for row in rows:
    #sleep(10) # Wait 10 sec, recommendations explained by [4] Miller, C. (2017)
    cells = row.findAll('td')
    if len(cells) > 0 :
        links = cells[1].findAll('a')
        for link in links:        
            extract_info_individual_event(link)
```


```python
"""
Once we have a list of dict in Events attributes, it's high time we defined
a panda dataframe to properly store the info.
It is done here and we present a glimpse of the data with the df.head() below.
"""



print(len(Events), ' eventos fueron añadidos')

df = pd.DataFrame(Events)
df = df[['event_name', 'weight_class', 'fighter_1', 'action', 'fighter_2', 'round', 'time', 'method' ]]

df.head(15)
```

    4869  eventos fueron añadidos





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
      <th>event_name</th>
      <th>weight_class</th>
      <th>fighter_1</th>
      <th>action</th>
      <th>fighter_2</th>
      <th>round</th>
      <th>time</th>
      <th>method</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>UFC Fight Night: Volkan vs. Smith</td>
      <td>Light Heavyweight</td>
      <td>Anthony Smith</td>
      <td>def.</td>
      <td>Volkan Oezdemir</td>
      <td>3</td>
      <td>4:26</td>
      <td>Submission (rear-naked choke)</td>
    </tr>
    <tr>
      <th>1</th>
      <td>UFC Fight Night: Volkan vs. Smith</td>
      <td>Catchweight (147 lbs)</td>
      <td>Michael Johnson</td>
      <td>def.</td>
      <td>Artem Lobov</td>
      <td>3</td>
      <td>5:00</td>
      <td>Decision (unanimous) (29-28, 29-28, 30-27)</td>
    </tr>
    <tr>
      <th>2</th>
      <td>UFC Fight Night: Volkan vs. Smith</td>
      <td>Light Heavyweight</td>
      <td>Misha Cirkunov</td>
      <td>def.</td>
      <td>Patrick Cummins</td>
      <td>1</td>
      <td>2:40</td>
      <td>Submission (arm-triangle choke)</td>
    </tr>
    <tr>
      <th>3</th>
      <td>UFC Fight Night: Volkan vs. Smith</td>
      <td>Bantamweight</td>
      <td>Andre Soukhamthath</td>
      <td>def.</td>
      <td>Jonathan Martinez</td>
      <td>3</td>
      <td>5:00</td>
      <td>Decision (unanimous) (30-26, 29-28, 29-28)</td>
    </tr>
    <tr>
      <th>4</th>
      <td>UFC Fight Night: Volkan vs. Smith</td>
      <td>Light Heavyweight</td>
      <td>Gian Villante</td>
      <td>def.</td>
      <td>Ed Herman</td>
      <td>3</td>
      <td>5:00</td>
      <td>Decision (split) (29-28, 28-29, 29-28)</td>
    </tr>
    <tr>
      <th>5</th>
      <td>UFC Fight Night: Volkan vs. Smith</td>
      <td>Welterweight</td>
      <td>Court McGee</td>
      <td>def.</td>
      <td>Alex Garcia</td>
      <td>3</td>
      <td>5:00</td>
      <td>Decision (unanimous) (29-28, 29-28, 30-28)</td>
    </tr>
    <tr>
      <th>6</th>
      <td>UFC Fight Night: Volkan vs. Smith</td>
      <td>Welterweight</td>
      <td>Sean Strickland</td>
      <td>def.</td>
      <td>Nordine Taleb</td>
      <td>2</td>
      <td>3:10</td>
      <td>TKO (punches)</td>
    </tr>
    <tr>
      <th>7</th>
      <td>UFC Fight Night: Volkan vs. Smith</td>
      <td>Lightweight</td>
      <td>Nasrat Haqparast</td>
      <td>def.</td>
      <td>Thibault Gouti</td>
      <td>3</td>
      <td>5:00</td>
      <td>Decision (unanimous) (29-27, 29-28, 30-26)</td>
    </tr>
    <tr>
      <th>8</th>
      <td>UFC Fight Night: Volkan vs. Smith</td>
      <td>Featherweight</td>
      <td>Calvin Kattar</td>
      <td>def.</td>
      <td>Chris Fishgold</td>
      <td>1</td>
      <td>4:11</td>
      <td>TKO (punches)</td>
    </tr>
    <tr>
      <th>9</th>
      <td>UFC Fight Night: Volkan vs. Smith</td>
      <td>Women's Bantamweight</td>
      <td>Talita Bernardo</td>
      <td>def.</td>
      <td>Sarah Moras</td>
      <td>3</td>
      <td>5:00</td>
      <td>Decision (unanimous) (30-27, 29-28, 29-28)</td>
    </tr>
    <tr>
      <th>10</th>
      <td>UFC Fight Night: Volkan vs. Smith</td>
      <td>Lightweight</td>
      <td>Don Madge</td>
      <td>def.</td>
      <td>Te'Jovan Edwards</td>
      <td>2</td>
      <td>0:14</td>
      <td>KO (head kick)</td>
    </tr>
    <tr>
      <th>11</th>
      <td>UFC Fight Night: Volkan vs. Smith</td>
      <td>Heavyweight</td>
      <td>Arjan Bhullar</td>
      <td>def.</td>
      <td>Marcelo Golm</td>
      <td>3</td>
      <td>5:00</td>
      <td>Decision (unanimous) (29-28, 29-28, 29-27)</td>
    </tr>
    <tr>
      <th>12</th>
      <td>UFC Fight Night: Volkan vs. Smith</td>
      <td>Lightweight</td>
      <td>Stevie Ray</td>
      <td>def.</td>
      <td>Jessin Ayari</td>
      <td>3</td>
      <td>5:00</td>
      <td>Decision (unanimous) (29-28, 29-28, 30-27)</td>
    </tr>
    <tr>
      <th>13</th>
      <td>UFC 229: Khabib vs. McGregor</td>
      <td>Lightweight</td>
      <td>Khabib Nurmagomedov</td>
      <td>def.</td>
      <td>Conor McGregor</td>
      <td>4</td>
      <td>3:03</td>
      <td>Submission (neck crank)</td>
    </tr>
    <tr>
      <th>14</th>
      <td>UFC 229: Khabib vs. McGregor</td>
      <td>Lightweight</td>
      <td>Tony Ferguson</td>
      <td>def.</td>
      <td>Anthony Pettis</td>
      <td>2</td>
      <td>5:00</td>
      <td>TKO (corner stoppage)</td>
    </tr>
  </tbody>
</table>
</div>




```python
"""
This is particularly a visual part of the presentation of the data.
For each event, I'm presenting the folder advertising it.
Remembering that we download each of these pictures in a method 
explained before
"""


df_imgs = df['event_name'].drop_duplicates()

print('Imagenes: ', len(df_imgs))
display(HTML('<table style="width:100%; border: 1px solid black;"><tr>'))
i = 0
row = ''
for img in df_imgs:
    img_src = 'pictures/' + img
    if i == 20:
        row = '{}<td><img src=\'{}\' alt=\'{}\' /></td></tr>'.format(row, img_src, img)
        display(HTML(row))
        row = '<tr>'
        i = 0
    else:
        row = '{}<td><img src=\'{}\' /></td>'.format(row, img_src)
     
    i += 1

display(HTML('</tr></table>'))


```

    Imagenes:  454



<table style="width:100%; border: 1px solid black;"><tr>



<td><img src='pictures/UFC Fight Night: Volkan vs. Smith' /></td><td><img src='pictures/UFC 229: Khabib vs. McGregor' /></td><td><img src='pictures/UFC Fight Night: Santos vs. Anders' /></td><td><img src='pictures/UFC Fight Night: Hunt vs. Oliynyk' /></td><td><img src='pictures/UFC 228: Woodley vs. Till' /></td><td><img src='pictures/UFC Fight Night: Gaethje vs. Vick' /></td><td><img src='pictures/UFC 227: Dillashaw vs. Garbrandt 2' /></td><td><img src='pictures/UFC on Fox: Alvarez vs. Poirier 2' /></td><td><img src='pictures/UFC Fight Night: Shogun vs. Smith' /></td><td><img src='pictures/UFC Fight Night: dos Santos vs. Ivanov' /></td><td><img src='pictures/UFC 226: Miocic vs. Cormier' /></td><td><img src='pictures/The Ultimate Fighter: Undefeated Finale' /></td><td><img src='pictures/UFC Fight Night: Cowboy vs. Edwards' /></td><td><img src='pictures/UFC 225: Whittaker vs. Romero 2' /></td><td><img src='pictures/UFC Fight Night: Rivera vs. Moraes' /></td><td><img src='pictures/UFC Fight Night: Thompson vs. Till' /></td><td><img src='pictures/UFC Fight Night: Maia vs. Usman' /></td><td><img src='pictures/UFC 224: Nunes vs. Pennington' /></td><td><img src='pictures/UFC Fight Night: Barboza vs. Lee' /></td><td><img src='pictures/UFC on Fox: Poirier vs. Gaethje' /></td><td><img src='pictures/UFC 223: Khabib vs. Iaquinta' alt='UFC 223: Khabib vs. Iaquinta' /></td></tr>



<tr><td><img src='pictures/UFC Fight Night: Werdum vs. Volkov' /></td><td><img src='pictures/UFC 222: Cyborg vs. Kunitskaya' /></td><td><img src='pictures/UFC on Fox: Emmett vs. Stephens' /></td><td><img src='pictures/UFC Fight Night: Cowboy vs. Medeiros' /></td><td><img src='pictures/UFC 221: Romero vs. Rockhold' /></td><td><img src='pictures/UFC Fight Night: Machida vs. Anders' /></td><td><img src='pictures/UFC on Fox: Jacaré vs. Brunson 2' /></td><td><img src='pictures/UFC 220: Miocic vs. Ngannou' /></td><td><img src='pictures/UFC Fight Night: Stephens vs. Choi' /></td><td><img src='pictures/UFC 219: Cyborg vs. Holm' /></td><td><img src='pictures/UFC on Fox: Lawler vs. dos Anjos' /></td><td><img src='pictures/UFC Fight Night: Swanson vs. Ortega' /></td><td><img src='pictures/UFC 218: Holloway vs. Aldo 2' /></td><td><img src='pictures/The Ultimate Fighter: A New World Champion Finale' /></td><td><img src='pictures/UFC Fight Night: Bisping vs. Gastelum' /></td><td><img src='pictures/UFC Fight Night: Werdum vs. Tybura' /></td><td><img src='pictures/UFC Fight Night: Poirier vs. Pettis' /></td><td><img src='pictures/UFC 217: Bisping vs. St-Pierre' /></td><td><img src='pictures/UFC Fight Night: Brunson vs. Machida' /></td><td><img src='pictures/UFC Fight Night: Cowboy vs. Till' alt='UFC Fight Night: Cowboy vs. Till' /></td></tr>



<tr><td><img src='pictures/UFC 216: Ferguson vs. Lee' /></td><td><img src='pictures/UFC Fight Night: Saint Preux vs. Okami' /></td><td><img src='pictures/UFC Fight Night: Rockhold vs. Branch' /></td><td><img src='pictures/UFC 215: Nunes vs. Shevchenko 2' /></td><td><img src='pictures/UFC Fight Night: Volkov vs. Struve' /></td><td><img src='pictures/UFC Fight Night: Pettis vs. Moreno' /></td><td><img src='pictures/UFC 214: Cormier vs. Jones 2' /></td><td><img src='pictures/UFC on Fox: Weidman vs. Gastelum' /></td><td><img src='pictures/UFC Fight Night: Nelson vs. Ponzinibbio' /></td><td><img src='pictures/UFC 213: Romero vs. Whittaker' /></td><td><img src='pictures/The Ultimate Fighter: Redemption Finale' /></td><td><img src='pictures/UFC Fight Night: Chiesa vs. Lee' /></td><td><img src='pictures/UFC Fight Night: Holm vs. Correia' /></td><td><img src='pictures/UFC Fight Night: Lewis vs. Hunt' /></td><td><img src='pictures/UFC 212: Aldo vs. Holloway' /></td><td><img src='pictures/UFC Fight Night: Gustafsson vs. Teixeira' /></td><td><img src='pictures/UFC 211: Miocic vs. dos Santos 2' /></td><td><img src='pictures/UFC Fight Night: Swanson vs. Lobov' /></td><td><img src='pictures/UFC on Fox: Johnson vs. Reis' /></td><td><img src='pictures/UFC 210: Cormier vs. Johnson 2' alt='UFC 210: Cormier vs. Johnson 2' /></td></tr>



<tr><td><img src='pictures/UFC Fight Night: Manuwa vs. Anderson' /></td><td><img src='pictures/UFC Fight Night: Belfort vs. Gastelum' /></td><td><img src='pictures/UFC 209: Woodley vs. Thompson 2' /></td><td><img src='pictures/UFC Fight Night: Lewis vs. Browne' /></td><td><img src='pictures/UFC 208: Holm vs. de Randamie' /></td><td><img src='pictures/UFC Fight Night: Bermudez vs. Korean Zombie' /></td><td><img src='pictures/UFC on Fox: Shevchenko vs. Peña' /></td><td><img src='pictures/UFC Fight Night: Rodríguez vs. Penn' /></td><td><img src='pictures/UFC 207: Nunes vs. Rousey' /></td><td><img src='pictures/UFC on Fox: VanZant vs. Waterson' /></td><td><img src='pictures/UFC 206: Holloway vs. Pettis' /></td><td><img src='pictures/UFC Fight Night: Lewis vs. Abdurakhimov' /></td><td><img src='pictures/The Ultimate Fighter: Tournament of Champions Finale' /></td><td><img src='pictures/UFC Fight Night: Whittaker vs. Brunson' /></td><td><img src='pictures/UFC Fight Night: Bader vs. Nogueira 2' /></td><td><img src='pictures/UFC Fight Night: Mousasi vs. Hall 2' /></td><td><img src='pictures/UFC 205: Alvarez vs. McGregor' /></td><td><img src='pictures/The Ultimate Fighter Latin America 3 Finale: dos Anjos vs. Ferguson' /></td><td><img src='pictures/UFC 204: Bisping vs. Henderson 2' /></td><td><img src='pictures/UFC Fight Night: Lineker vs. Dodson' alt='UFC Fight Night: Lineker vs. Dodson' /></td></tr>



<tr><td><img src='pictures/UFC Fight Night: Cyborg vs. Lansberg' /></td><td><img src='pictures/UFC Fight Night: Poirier vs. Johnson' /></td><td><img src='pictures/UFC 203: Miocic vs. Overeem' /></td><td><img src='pictures/UFC Fight Night: Arlovski vs. Barnett' /></td><td><img src='pictures/UFC on Fox: Maia vs. Condit' /></td><td><img src='pictures/UFC 202: Diaz vs. McGregor 2' /></td><td><img src='pictures/UFC Fight Night: Rodríguez vs. Caceres' /></td><td><img src='pictures/UFC 201: Lawler vs. Woodley' /></td><td><img src='pictures/UFC on Fox: Holm vs. Shevchenko' /></td><td><img src='pictures/UFC Fight Night: McDonald vs. Lineker' /></td><td><img src='pictures/UFC 200: Tate vs. Nunes' /></td><td><img src='pictures/The Ultimate Fighter: Team Joanna vs. Team Cláudia Finale' /></td><td><img src='pictures/UFC Fight Night: dos Anjos vs. Alvarez' /></td><td><img src='pictures/UFC Fight Night: MacDonald vs. Thompson' /></td><td><img src='pictures/UFC 199: Rockhold vs. Bisping 2' /></td><td><img src='pictures/UFC Fight Night: Almeida vs. Garbrandt' /></td><td><img src='pictures/UFC 198: Werdum vs. Miocic' /></td><td><img src='pictures/UFC Fight Night: Overeem vs. Arlovski' /></td><td><img src='pictures/UFC 197: Jones vs. Saint Preux' /></td><td><img src='pictures/UFC on Fox: Teixeira vs. Evans' alt='UFC on Fox: Teixeira vs. Evans' /></td></tr>



<tr><td><img src='pictures/UFC Fight Night: Rothwell vs. dos Santos' /></td><td><img src='pictures/UFC Fight Night: Hunt vs. Mir' /></td><td><img src='pictures/UFC 196: McGregor vs. Diaz' /></td><td><img src='pictures/UFC Fight Night: Silva vs. Bisping' /></td><td><img src='pictures/UFC Fight Night: Cowboy vs. Cowboy' /></td><td><img src='pictures/UFC Fight Night: Hendricks vs. Thompson' /></td><td><img src='pictures/UFC on Fox: Johnson vs. Bader' /></td><td><img src='pictures/UFC Fight Night: Dillashaw vs. Cruz' /></td><td><img src='pictures/UFC 195: Lawler vs. Condit' /></td><td><img src='pictures/UFC on Fox: dos Anjos vs. Cerrone 2' /></td><td><img src='pictures/UFC 194: Aldo vs. McGregor' /></td><td><img src='pictures/The Ultimate Fighter: Team McGregor vs. Team Faber Finale' /></td><td><img src='pictures/UFC Fight Night: Namajunas vs. VanZant' /></td><td><img src='pictures/UFC Fight Night: Henderson vs. Masvidal' /></td><td><img src='pictures/The Ultimate Fighter Latin America 2 Finale: Magny vs. Gastelum' /></td><td><img src='pictures/UFC 193: Rousey vs. Holm' /></td><td><img src='pictures/UFC Fight Night: Belfort vs. Henderson 3' /></td><td><img src='pictures/UFC Fight Night: Holohan vs. Smolka' /></td><td><img src='pictures/UFC 192: Cormier vs. Gustafsson' /></td><td><img src='pictures/UFC Fight Night: Barnett vs. Nelson' alt='UFC Fight Night: Barnett vs. Nelson' /></td></tr>



<tr><td><img src='pictures/UFC 191: Johnson vs. Dodson 2' /></td><td><img src='pictures/UFC Fight Night: Holloway vs. Oliveira' /></td><td><img src='pictures/UFC Fight Night: Teixeira vs. Saint Preux' /></td><td><img src='pictures/UFC 190: Rousey vs. Correia' /></td><td><img src='pictures/UFC on Fox: Dillashaw vs. Barão 2' /></td><td><img src='pictures/UFC Fight Night: Bisping vs. Leites' /></td><td><img src='pictures/UFC Fight Night: Mir vs. Duffee' /></td><td><img src='pictures/The Ultimate Fighter: American Top Team vs. Blackzilians Finale' /></td><td><img src='pictures/UFC 189: Mendes vs. McGregor' /></td><td><img src='pictures/UFC Fight Night: Machida vs. Romero' /></td><td><img src='pictures/UFC Fight Night: Jędrzejczyk vs. Penne' /></td><td><img src='pictures/UFC 188: Velasquez vs. Werdum' /></td><td><img src='pictures/UFC Fight Night: Boetsch vs. Henderson' /></td><td><img src='pictures/UFC Fight Night: Condit vs. Alves' /></td><td><img src='pictures/UFC 187: Johnson vs. Cormier' /></td><td><img src='pictures/UFC Fight Night: Edgar vs. Faber' /></td><td><img src='pictures/UFC Fight Night: Miocic vs. Hunt' /></td><td><img src='pictures/UFC 186: Johnson vs. Horiguchi' /></td><td><img src='pictures/UFC on Fox: Machida vs. Rockhold' /></td><td><img src='pictures/UFC Fight Night: Gonzaga vs. Cro Cop 2' alt='UFC Fight Night: Gonzaga vs. Cro Cop 2' /></td></tr>



<tr><td><img src='pictures/UFC Fight Night: Mendes vs. Lamas' /></td><td><img src='pictures/UFC Fight Night: Maia vs. LaFlare' /></td><td><img src='pictures/UFC 185: Pettis vs. dos Anjos' /></td><td><img src='pictures/UFC 184: Rousey vs. Zingano' /></td><td><img src='pictures/UFC Fight Night: Bigfoot vs. Mir' /></td><td><img src='pictures/UFC Fight Night: Henderson vs. Thatch' /></td><td><img src='pictures/UFC 183: Silva vs. Diaz' /></td><td><img src='pictures/UFC on Fox: Gustafsson vs. Johnson' /></td><td><img src='pictures/UFC Fight Night: McGregor vs. Siver' /></td><td><img src='pictures/UFC 182: Jones vs. Cormier' /></td><td><img src='pictures/UFC Fight Night: Machida vs. Dollaway' /></td><td><img src='pictures/UFC on Fox: dos Santos vs. Miocic' /></td><td><img src='pictures/The Ultimate Fighter: A Champion Will Be Crowned Finale' /></td><td><img src='pictures/UFC 181: Hendricks vs. Lawler II' /></td><td><img src='pictures/UFC Fight Night: Edgar vs. Swanson' /></td><td><img src='pictures/UFC 180: Werdum vs. Hunt' /></td><td><img src='pictures/UFC Fight Night: Shogun vs. Saint Preux' /></td><td><img src='pictures/UFC Fight Night: Rockhold vs. Bisping' /></td><td><img src='pictures/UFC 179: Aldo vs. Mendes 2' /></td><td><img src='pictures/UFC Fight Night: MacDonald vs. Saffiedine' alt='UFC Fight Night: MacDonald vs. Saffiedine' /></td></tr>



<tr><td><img src='pictures/UFC Fight Night: Nelson vs. Story' /></td><td><img src='pictures/UFC 178: Johnson vs. Cariaso' /></td><td><img src='pictures/UFC Fight Night: Hunt vs. Nelson' /></td><td><img src='pictures/UFC Fight Night: Bigfoot vs. Arlovski' /></td><td><img src='pictures/UFC Fight Night: Jacaré vs. Mousasi' /></td><td><img src='pictures/UFC 177: Dillashaw vs. Soto' /></td><td><img src='pictures/UFC Fight Night: Henderson vs. dos Anjos' /></td><td><img src='pictures/UFC Fight Night: Bisping vs. Le' /></td><td><img src='pictures/UFC Fight Night: Bader vs. Saint Preux' /></td><td><img src='pictures/UFC on Fox: Lawler vs. Brown' /></td><td><img src='pictures/UFC Fight Night: McGregor vs. Brandao' /></td><td><img src='pictures/UFC Fight Night: Cerrone vs. Miller' /></td><td><img src='pictures/The Ultimate Fighter: Team Edgar vs. Team Penn Finale' /></td><td><img src='pictures/UFC 175: Weidman vs. Machida' /></td><td><img src='pictures/UFC Fight Night: Swanson vs. Stephens' /></td><td><img src='pictures/UFC Fight Night: Te Huna vs. Marquardt' /></td><td><img src='pictures/UFC 174: Johnson vs. Bagautinov' /></td><td><img src='pictures/UFC Fight Night: Henderson vs. Khabilov' /></td><td><img src='pictures/The Ultimate Fighter Brazil 3 Finale: Miocic vs. Maldonado' /></td><td><img src='pictures/UFC Fight Night: Muñoz vs. Mousasi' alt='UFC Fight Night: Muñoz vs. Mousasi' /></td></tr>



<tr><td><img src='pictures/UFC 173: Barão vs. Dillashaw' /></td><td><img src='pictures/UFC Fight Night: Brown vs. Silva' /></td><td><img src='pictures/UFC 172: Jones vs. Teixeira' /></td><td><img src='pictures/UFC on Fox: Werdum vs. Browne' /></td><td><img src='pictures/The Ultimate Fighter Nations Finale: Bisping vs. Kennedy' /></td><td><img src='pictures/UFC Fight Night: Nogueira vs. Nelson' /></td><td><img src='pictures/UFC Fight Night: Shogun vs. Henderson 2' /></td><td><img src='pictures/UFC 171: Hendricks vs. Lawler' /></td><td><img src='pictures/UFC Fight Night: Gustafsson vs. Manuwa' /></td><td><img src='pictures/The Ultimate Fighter China Finale: Kim vs. Hathaway' /></td><td><img src='pictures/UFC 170: Rousey vs. McMann' /></td><td><img src='pictures/UFC Fight Night: Machida vs. Mousasi' /></td><td><img src='pictures/UFC 169: Barão vs. Faber II' /></td><td><img src='pictures/UFC on Fox: Henderson vs. Thomson' /></td><td><img src='pictures/UFC Fight Night: Rockhold vs. Philippou' /></td><td><img src='pictures/UFC Fight Night: Saffiedine vs. Lim' /></td><td><img src='pictures/UFC 168: Weidman vs. Silva 2' /></td><td><img src='pictures/UFC on Fox: Johnson vs. Benavidez 2' /></td><td><img src='pictures/UFC Fight Night: Hunt vs. Bigfoot' /></td><td><img src='pictures/The Ultimate Fighter: Team Rousey vs. Team Tate Finale' alt='The Ultimate Fighter: Team Rousey vs. Team Tate Finale' /></td></tr>



<tr><td><img src='pictures/UFC 167: St-Pierre vs. Hendricks' /></td><td><img src='pictures/UFC Fight Night: Belfort vs. Henderson' /></td><td><img src='pictures/UFC: Fight for the Troops 3' /></td><td><img src='pictures/UFC Fight Night: Machida vs. Muñoz' /></td><td><img src='pictures/UFC 166: Velasquez vs. dos Santos III' /></td><td><img src='pictures/UFC Fight Night: Maia vs. Shields' /></td><td><img src='pictures/UFC 165: Jones vs. Gustafsson' /></td><td><img src='pictures/UFC Fight Night: Teixeira vs. Bader' /></td><td><img src='pictures/UFC 164: Henderson vs. Pettis' /></td><td><img src='pictures/UFC Fight Night: Condit vs. Kampmann 2' /></td><td><img src='pictures/UFC Fight Night: Shogun vs. Sonnen' /></td><td><img src='pictures/UFC 163: Aldo vs. Korean Zombie' /></td><td><img src='pictures/UFC on Fox: Johnson vs. Moraga' /></td><td><img src='pictures/UFC 162: Silva vs. Weidman' /></td><td><img src='pictures/UFC 161: Evans vs. Henderson' /></td><td><img src='pictures/UFC on Fuel TV: Nogueira vs. Werdum' /></td><td><img src='pictures/UFC 160: Velasquez vs. Bigfoot 2' /></td><td><img src='pictures/UFC on FX: Belfort vs. Rockhold' /></td><td><img src='pictures/UFC 159: Jones vs. Sonnen' /></td><td><img src='pictures/UFC on Fox: Henderson vs. Melendez' alt='UFC on Fox: Henderson vs. Melendez' /></td></tr>



<tr><td><img src='pictures/The Ultimate Fighter: Team Jones vs. Team Sonnen Finale' /></td><td><img src='pictures/UFC on Fuel TV: Mousasi vs. Latifi' /></td><td><img src='pictures/UFC 158: St-Pierre vs. Diaz' /></td><td><img src='pictures/UFC on Fuel TV: Silva vs. Stann' /></td><td><img src='pictures/UFC 157: Rousey vs. Carmouche' /></td><td><img src='pictures/UFC on Fuel TV: Barão vs. McDonald' /></td><td><img src='pictures/UFC 156: Aldo vs. Edgar' /></td><td><img src='pictures/UFC on Fox: Johnson vs. Dodson' /></td><td><img src='pictures/UFC on FX: Belfort vs. Bisping' /></td><td><img src='pictures/UFC 155: dos Santos vs. Velasquez 2' /></td><td><img src='pictures/The Ultimate Fighter: Team Carwin vs. Team Nelson Finale' /></td><td><img src='pictures/UFC on FX: Sotiropoulos vs. Pearson' /></td><td><img src='pictures/UFC on Fox: Henderson vs. Diaz' /></td><td><img src='pictures/UFC 154: St-Pierre vs. Condit' /></td><td><img src='pictures/UFC on Fuel TV: Franklin vs. Le' /></td><td><img src='pictures/UFC 153: Silva vs. Bonnar' /></td><td><img src='pictures/UFC on FX: Browne vs. Bigfoot' /></td><td><img src='pictures/UFC on Fuel TV: Struve vs. Miocic' /></td><td><img src='pictures/UFC 152: Jones vs. Belfort' /></td><td><img src='pictures/UFC 150: Henderson vs. Edgar II' alt='UFC 150: Henderson vs. Edgar II' /></td></tr>



<tr><td><img src='pictures/UFC on Fox: Shogun vs. Vera' /></td><td><img src='pictures/UFC 149: Faber vs. Barão' /></td><td><img src='pictures/UFC on Fuel TV: Muñoz vs. Weidman' /></td><td><img src='pictures/UFC 148: Silva vs. Sonnen II' /></td><td><img src='pictures/UFC 147: Silva vs. Franklin II' /></td><td><img src='pictures/UFC on FX: Maynard vs. Guida' /></td><td><img src='pictures/UFC on FX: Johnson vs. McCall' /></td><td><img src='pictures/The Ultimate Fighter: Live Finale' /></td><td><img src='pictures/UFC 146: dos Santos vs. Mir' /></td><td><img src='pictures/UFC on Fuel TV: Korean Zombie vs. Poirier' /></td><td><img src='pictures/UFC on Fox: Diaz vs. Miller' /></td><td><img src='pictures/UFC 145: Jones vs. Evans' /></td><td><img src='pictures/UFC on Fuel TV: Gustafsson vs. Silva' /></td><td><img src='pictures/UFC on FX: Alves vs. Kampmann' /></td><td><img src='pictures/UFC 144: Edgar vs. Henderson' /></td><td><img src='pictures/UFC on Fuel TV: Sanchez vs. Ellenberger' /></td><td><img src='pictures/UFC 143: Diaz vs. Condit' /></td><td><img src='pictures/UFC on Fox: Evans vs. Davis' /></td><td><img src='pictures/UFC on FX: Guillard vs. Miller' /></td><td><img src='pictures/UFC 142: Aldo vs. Mendes' alt='UFC 142: Aldo vs. Mendes' /></td></tr>



<tr><td><img src='pictures/UFC 141: Lesnar vs. Overeem' /></td><td><img src='pictures/UFC 140: Jones vs. Machida' /></td><td><img src='pictures/The Ultimate Fighter: Team Bisping vs. Team Miller Finale' /></td><td><img src='pictures/UFC 139: Shogun vs. Henderson' /></td><td><img src='pictures/UFC on Fox: Velasquez vs. dos Santos' /></td><td><img src='pictures/UFC 138: Leben vs. Muñoz' /></td><td><img src='pictures/UFC 137: Penn vs. Diaz' /></td><td><img src='pictures/UFC 136: Edgar vs. Maynard III' /></td><td><img src='pictures/UFC Live: Cruz vs. Johnson' /></td><td><img src='pictures/UFC 135: Jones vs. Rampage' /></td><td><img src='pictures/UFC Fight Night: Shields vs. Ellenberger' /></td><td><img src='pictures/UFC 134: Silva vs. Okami' /></td><td><img src='pictures/UFC Live: Hardy vs. Lytle' /></td><td><img src='pictures/UFC 133: Evans vs. Ortiz' /></td><td><img src='pictures/UFC 132: Cruz vs. Faber' /></td><td><img src='pictures/UFC Live: Kongo vs. Barry' /></td><td><img src='pictures/UFC 131: dos Santos vs. Carwin' /></td><td><img src='pictures/The Ultimate Fighter: Team Lesnar vs. Team dos Santos Finale' /></td><td><img src='pictures/UFC 130: Rampage vs. Hamill' /></td><td><img src='pictures/UFC 129: St-Pierre vs. Shields' alt='UFC 129: St-Pierre vs. Shields' /></td></tr>



<tr><td><img src='pictures/UFC Fight Night: Nogueira vs. Davis' /></td><td><img src='pictures/UFC 128: Shogun vs. Jones' /></td><td><img src='pictures/UFC Live: Sanchez vs. Kampmann' /></td><td><img src='pictures/UFC 127: Penn vs. Fitch' /></td><td><img src='pictures/UFC 126: Silva vs. Belfort' /></td><td><img src='pictures/UFC: Fight for the Troops 2' /></td><td><img src='pictures/UFC 125: Resolution' /></td><td><img src='pictures/UFC 124: St-Pierre vs. Koscheck 2' /></td><td><img src='pictures/The Ultimate Fighter: Team GSP vs. Team Koscheck Finale' /></td><td><img src='pictures/UFC 123: Rampage vs. Machida' /></td><td><img src='pictures/UFC 122: Marquardt vs. Okami' /></td><td><img src='pictures/UFC 121: Lesnar vs. Velasquez' /></td><td><img src='pictures/UFC 120: Bisping vs. Akiyama' /></td><td><img src='pictures/UFC 119: Mir vs. Cro Cop' /></td><td><img src='pictures/UFC Fight Night: Marquardt vs. Palhares' /></td><td><img src='pictures/UFC 118: Edgar vs. Penn 2' /></td><td><img src='pictures/UFC 117: Silva vs. Sonnen' /></td><td><img src='pictures/UFC Live: Jones vs. Matyushenko' /></td><td><img src='pictures/UFC 116: Lesnar vs. Carwin' /></td><td><img src='pictures/The Ultimate Fighter: Team Liddell vs. Team Ortiz Finale' alt='The Ultimate Fighter: Team Liddell vs. Team Ortiz Finale' /></td></tr>



<tr><td><img src='pictures/UFC 115: Liddell vs. Franklin' /></td><td><img src='pictures/UFC 114: Rampage vs. Evans' /></td><td><img src='pictures/UFC 113: Machida vs. Shogun 2' /></td><td><img src='pictures/UFC 112: Invincible' /></td><td><img src='pictures/UFC Fight Night: Florian vs. Gomi' /></td><td><img src='pictures/UFC 111: St-Pierre vs. Hardy' /></td><td><img src='pictures/UFC Live: Vera vs. Jones' /></td><td><img src='pictures/UFC 110: Nogueira vs. Velasquez' /></td><td><img src='pictures/UFC 109: Relentless' /></td><td><img src='pictures/UFC Fight Night: Maynard vs. Diaz' /></td><td><img src='pictures/UFC 108: Evans vs. Silva' /></td><td><img src='pictures/UFC 107: Penn vs. Sanchez' /></td><td><img src='pictures/The Ultimate Fighter: Heavyweights Finale' /></td><td><img src='pictures/UFC 106: Ortiz vs. Griffin 2' /></td><td><img src='pictures/UFC 105: Couture vs. Vera' /></td><td><img src='pictures/UFC 104: Machida vs. Shogun' /></td><td><img src='pictures/UFC 103: Franklin vs. Belfort' /></td><td><img src='pictures/UFC Fight Night: Diaz vs. Guillard' /></td><td><img src='pictures/UFC 102: Couture vs. Nogueira' /></td><td><img src='pictures/UFC 101: Declaration' alt='UFC 101: Declaration' /></td></tr>



<tr><td><img src='pictures/UFC 100' /></td><td><img src='pictures/The Ultimate Fighter: United States vs. United Kingdom Finale' /></td><td><img src='pictures/UFC 99: The Comeback' /></td><td><img src='pictures/UFC 98: Evans vs. Machida' /></td><td><img src='pictures/UFC 97: Redemption' /></td><td><img src='pictures/UFC Fight Night: Condit vs. Kampmann' /></td><td><img src='pictures/UFC 96: Jackson vs. Jardine' /></td><td><img src='pictures/UFC 95: Sanchez vs. Stevenson' /></td><td><img src='pictures/UFC Fight Night: Lauzon vs. Stephens' /></td><td><img src='pictures/UFC 94: St-Pierre vs. Penn 2' /></td><td><img src='pictures/UFC 93: Franklin vs. Henderson' /></td><td><img src='pictures/UFC 92: The Ultimate 2008' /></td><td><img src='pictures/The Ultimate Fighter: Team Nogueira vs. Team Mir Finale' /></td><td><img src='pictures/UFC: Fight for the Troops' /></td><td><img src='pictures/UFC 91: Couture vs. Lesnar' /></td><td><img src='pictures/UFC 90: Silva vs. Côté' /></td><td><img src='pictures/UFC 89: Bisping vs. Leben' /></td><td><img src='pictures/UFC Fight Night: Diaz vs. Neer' /></td><td><img src='pictures/UFC 88: Breakthrough' /></td><td><img src='pictures/UFC 87: Seek and Destroy' alt='UFC 87: Seek and Destroy' /></td></tr>



<tr><td><img src='pictures/UFC Fight Night: Silva vs. Irvin' /></td><td><img src='pictures/UFC 86: Jackson vs. Griffin' /></td><td><img src='pictures/The Ultimate Fighter: Team Rampage vs. Team Forrest Finale' /></td><td><img src='pictures/UFC 85: Bedlam' /></td><td><img src='pictures/UFC 84: Ill Will' /></td><td><img src='pictures/UFC 83: Serra vs. St-Pierre 2' /></td><td><img src='pictures/UFC Fight Night: Florian vs. Lauzon' /></td><td><img src='pictures/UFC 82: Pride of a Champion' /></td><td><img src='pictures/UFC 81: Breaking Point' /></td><td><img src='pictures/UFC Fight Night: Swick vs. Burkman' /></td><td><img src='pictures/UFC 80: Rapid Fire' /></td><td><img src='pictures/UFC 79: Nemesis' /></td><td><img src='pictures/The Ultimate Fighter: Team Hughes vs. Team Serra Finale' /></td><td><img src='pictures/UFC 78: Validation' /></td><td><img src='pictures/UFC 77: Hostile Territory' /></td><td><img src='pictures/UFC 76: Knockout' /></td><td><img src='pictures/UFC Fight Night: Thomas vs. Florian' /></td><td><img src='pictures/UFC 75: Champion vs. Champion' /></td><td><img src='pictures/UFC 74: Respect' /></td><td><img src='pictures/UFC 73: Stacked' alt='UFC 73: Stacked' /></td></tr>



<tr><td><img src='pictures/The Ultimate Fighter: Team Pulver vs. Team Penn Finale' /></td><td><img src='pictures/UFC 72: Victory' /></td><td><img src='pictures/UFC Fight Night: Stout vs. Fisher' /></td><td><img src='pictures/UFC 71: Liddell vs. Jackson' /></td><td><img src='pictures/UFC 70: Nations Collide' /></td><td><img src='pictures/UFC 69: Shootout' /></td><td><img src='pictures/UFC Fight Night: Stevenson vs. Guillard' /></td><td><img src='pictures/UFC 68: The Uprising' /></td><td><img src='pictures/UFC 67: All or Nothing' /></td><td><img src='pictures/UFC Fight Night: Evans vs. Salmon' /></td><td><img src='pictures/UFC 66: Liddell vs. Ortiz' /></td><td><img src='pictures/UFC Fight Night: Sanchez vs. Riggs' /></td><td><img src='pictures/UFC 65: Bad Intentions' /></td><td><img src='pictures/The Ultimate Fighter: The Comeback Finale' /></td><td><img src='pictures/UFC 64: Unstoppable' /></td><td><img src='pictures/UFC Fight Night 6.5' /></td><td><img src='pictures/UFC 63: Hughes vs. Penn' /></td><td><img src='pictures/UFC 62: Liddell vs. Sobral' /></td><td><img src='pictures/UFC Fight Night 6' /></td><td><img src='pictures/UFC 61: Bitter Rivals' alt='UFC 61: Bitter Rivals' /></td></tr>



<tr><td><img src='pictures/UFC Ultimate Fight Night 5' /></td><td><img src='pictures/The Ultimate Fighter: Team Ortiz vs. Team Shamrock Finale' /></td><td><img src='pictures/UFC 60: Hughes vs. Gracie' /></td><td><img src='pictures/UFC 59: Reality Check' /></td><td><img src='pictures/UFC Ultimate Fight Night 4' /></td><td><img src='pictures/UFC 58: USA vs. Canada' /></td><td><img src='pictures/UFC 57: Liddell vs. Couture 3' /></td><td><img src='pictures/UFC Ultimate Fight Night 3' /></td><td><img src='pictures/UFC 56: Full Force' /></td><td><img src='pictures/The Ultimate Fighter: Team Hughes vs. Team Franklin Finale' /></td><td><img src='pictures/UFC 55: Fury' /></td><td><img src='pictures/UFC Ultimate Fight Night 2' /></td><td><img src='pictures/UFC 54: Boiling Point' /></td><td><img src='pictures/UFC Ultimate Fight Night' /></td><td><img src='pictures/UFC 53: Heavy Hitters' /></td><td><img src='pictures/UFC 52: Couture vs. Liddell' /></td><td><img src='pictures/The Ultimate Fighter: Team Couture vs. Team Liddell Finale' /></td><td><img src='pictures/UFC 51: Super Saturday' /></td><td><img src='pictures/UFC 50: The War of '04' /></td><td><img src='pictures/UFC 49: Unfinished Business' alt='UFC 49: Unfinished Business' /></td></tr>



<tr><td><img src='pictures/UFC 48: Payback' /></td><td><img src='pictures/UFC 47: It's On!' /></td><td><img src='pictures/UFC 46: Supernatural' /></td><td><img src='pictures/UFC 45: Revolution' /></td><td><img src='pictures/UFC 44: Undisputed' /></td><td><img src='pictures/UFC 43: Meltdown' /></td><td><img src='pictures/UFC 42: Sudden Impact' /></td><td><img src='pictures/UFC 41: Onslaught' /></td><td><img src='pictures/UFC 40: Vendetta' /></td><td><img src='pictures/UFC 39: The Warriors Return' /></td><td><img src='pictures/UFC 38: Brawl at the Hall' /></td><td><img src='pictures/UFC 37.5: As Real As It Gets' /></td><td><img src='pictures/UFC 37: High Impact' /></td><td><img src='pictures/UFC 36: Worlds Collide' /></td><td><img src='pictures/UFC 35: Throwdown' /></td><td><img src='pictures/UFC 34: High Voltage' /></td><td><img src='pictures/UFC 33: Victory in Vegas' /></td><td><img src='pictures/UFC 32: Showdown in the Meadowlands' /></td><td><img src='pictures/UFC 31: Locked and Loaded' /></td><td><img src='pictures/UFC 30: Battle on the Boardwalk' alt='UFC 30: Battle on the Boardwalk' /></td></tr>



<tr><td><img src='pictures/UFC 29: Defense of the Belts' /></td><td><img src='pictures/UFC 28: High Stakes' /></td><td><img src='pictures/UFC 27: Ultimate Bad Boyz' /></td><td><img src='pictures/UFC 26: Ultimate Field of Dreams' /></td><td><img src='pictures/UFC 25: Ultimate Japan 3' /></td><td><img src='pictures/UFC 24: First Defense' /></td><td><img src='pictures/UFC 23: Ultimate Japan 2' /></td><td><img src='pictures/UFC 22: Only One Can be Champion' /></td><td><img src='pictures/UFC 21: Return of the Champions' /></td><td><img src='pictures/UFC 20: Battle for the Gold' /></td><td><img src='pictures/UFC 19: Ultimate Young Guns' /></td><td><img src='pictures/UFC 18: The Road to the Heavyweight Title' /></td><td><img src='pictures/UFC Brazil: Ultimate Brazil' /></td><td><img src='pictures/UFC 17: Redemption' /></td><td><img src='pictures/UFC 16: Battle in the Bayou' /></td><td><img src='pictures/UFC Japan: Ultimate Japan' /></td><td><img src='pictures/UFC 15: Collision Course' /></td><td><img src='pictures/UFC 14: Showdown' /></td><td><img src='pictures/UFC 13: The Ultimate Force' /></td><td><img src='pictures/UFC 12: Judgement Day' alt='UFC 12: Judgement Day' /></td></tr>



</tr></table>


### Dataset: Dataset en formato CSV
Por aquí generamos el dataset que podrá ser consultado en el repositorio.


```python
"""
Last but not least, we are storing the dataframe in a CSV file
"""

file_name = 'ufc-events.csv'
df.to_csv(file_name)
```

### Referencias

<p style="text-align: justify">
[1] <b>Subirats, L., Calvo, M. (2018).</b> "<i>Web Scraping</i>". Editorial UOC. Barcelona: Universitat Autònoma de Barcelona.<p>
<p style="text-align: justify">
[2] <b>Masip, D. (?).</b> "<i>El lenguaje Python</i>". Editorial UOC. Barcelona: Universitat Autònoma de Barcelona.<p>
<p style="text-align: justify">
[3] <b>Lawson, R. (2015).</b> "<i>Web Scraping with Python</i>". Packt Publishing Ltd. Chapter 2. Scraping the Data.<p>
<p style="text-align: justify">
[4] <b>Miller, C. (2017).</b> "<i>Data Acquisition and Manipulation with Python - Acquire, Analyse, and Play with Data</i>". Packt Publishing Ltd. Chapter 2. Web Scraping with BeautifulSoup.<p>
    
<p style="text-align: justify">
[5] <b>Chhibber, A. (2018).</b> "<i>Web Scraping Using Python</i>". Technics Publications.<p>
