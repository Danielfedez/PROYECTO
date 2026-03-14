# 1. ¿Qué información puedo obtener del *dataset*

Para acceder al documento con el código, clica [aquí](Pregunta_1_code.ipynb)

Lo primero que hacemos es importar las librerías que nos acompañarán en este y posteriores análisis. 

```
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
from datasets import load_dataset`
```


Inspeccionando el *dataset* identificamos 785741 ofertas de empleo publicadas internacionalmente, y de cada oferta se nos indican 17 características (nombre del rol, país de origen de la oferta, si es presencial o en remoto, fecha de publicación...)




![dataset_info](Imágenes\dataset_info.png)


Podemos sacar algunas conclusiones preliminares sobre la naturaleza de los puestos ofrecidos. 





## Qué roles son los más anunciados

Comparamos los distintos roles con su respectivo número de ofertas. Definimos la variable `job_counts` como el recuento sobre la columna `job_title_short`. Representamos gráficamente con `seaborn`. 

```
dataset = load_dataset("lukebarousse/data_jobs")
df = dataset["train"].to_pandas()

job_counts = df["job_title_short"].value_counts()

job_counts = job_counts.reset_index(name="job_counts")

total = job_counts["job_counts"].sum()

job_counts["job_perc"] = job_counts["job_counts"]/total*100


sns.set_theme(style='ticks')
ax = sns.barplot(data=job_counts, x='job_counts', y='job_title_short', hue="job_counts", palette='dark:b_r', legend=False)
sns.despine()
plt.title('Top 10 roles con mayor número de ofertas ')
plt.xlabel('Número de ofertas publicadas')
plt.ylabel('')

for n, (count, perc) in enumerate(zip(job_counts['job_counts'], job_counts['job_perc'])):
    ax.text(count + 5000, n, f'{perc:.1f}%', va='center')
plt.show()
``` 

![Top_10_roles](Imágenes\Top_10_roles.png)

Se observa una clara brecha entre los tres puestos con mayor demanda y el resto, representando estos primeros el 70,6% del total de ofertas publicadas. Es un resultado esperable ya que el núcleo del ecosistema de datos actual está formado principalmente por estos roles. 

Es de suponer que las ofertas más abundantes corresponden a perfiles *junior/intermediate*, ya que existen roles específicamente *senior*, con mucha menos demanda (14,1% respecto al total) a causa de la menor necesidad de posiciones de liderazgo. 


Roles como *Business Analyst* o *Software Engineer* rondan el 5%-6%, mientros otros como *Machine Learning* y *Cloud Engineer* suman en conjunto el 3,4% del total, posiblemente debido a que son nichos más especializados. 

## A qué países pertenecen estas ofertas

Definimos `job_country` como los 20 países con mayor número de ofertas publicadas.

```
job_country = df["job_country"].value_counts().head(20).reset_index(name="country_counts")

sns.set_theme(style='ticks')
sns.barplot(data=job_country, x='country_counts', y="job_country", hue="country_counts", palette='dark:b_r', legend=False)
sns.despine()
plt.title('Número de puestos en función del país')
plt.xlabel('Número de ofertas publicadas')
plt.ylabel('')
plt.show()
```


![ofertas_paises](Imágenes\ofertas_paises.png)

Estados Unidos lidera la lista de países con mayor demanda en ciencia de datos, siendo el epicentro de las grandes tecnológicas (*Big Tech*) y el desarrollo de IA. Siguiéndole está India, posiblemente por ser el principal destino de *outsourcing* para análisis de datos y soporte de ingeniería a gran escala. 


## Qué empresas publican más ofertas

Definimos `top_companies` y filtramos las 15 empresas con mayor oferta laboral.

```
top_companies = df["company_name"].value_counts().head(15).reset_index(name="company_counts")

sns.set_theme(style='ticks')
sns.barplot(data=top_companies, x='company_counts', y="company_name", hue="company_counts", palette='dark:b_r', legend=False)
sns.despine()
plt.title('Número de puestos en función de la compañía')
plt.xlabel('Número de ofertas publicadas')
plt.ylabel('')
plt.show()
```
![puestos_por_empresa](Imágenes\puestos_por_empresa.png)

Analizando los cinco primeros puestos cobra más relevancia el *outsourcing* y reclutamiento externo comentado anteriormente. No aparecen gigantes tecnológicos como Google o Meta, sino empresas cuyo modelo de negocio es encontrar talento para terceros. De igual forma, pero a la cola de la lista, aparecen Accenture i Deloitte, dos de las "*Big Four*".

Algunas empresas, como Citi o Capital One, representan al sector bancario, y otras como Walmart o UnitedHealth Group, cadenas de suministro o aseguraduras médicas, respectivamente. Se trata de compañías con gran alcance y con alta demanda de analistas. 


## Algunas características de los puestos

En esta sección analizaremos algunas condiciones de los puestos ofertados: si son presenciales o remotos, si se requiere alguna carrera para poder acceder al puesto, y si incluyen seguro médico o no.

Para visualizar estos datos usamos tres gráficos circulares, uno para cada condición laboral. Creamos un diccionario para poder unificar en un solo código las tres figuras.

```
dict_column = {
    'job_work_from_home': 'Opción de teletrabajo',
    'job_no_degree_mention': 'Se requiere carrera',
    'job_health_insurance': 'Incluye seguro médico'
}

fig, ax = plt.subplots(1, 3, figsize=(11, 3.5))

for i, (column, title) in enumerate(dict_column.items()):
    ax[i].pie(df[column].value_counts(), labels=['No', 'Sí'], autopct='%1.1f%%', startangle=90)
    ax[i].set_title(title)

plt.show()
```


![condiciones_trabajo](Imágenes\condiciones_trabajo.png)

Se observa que, pese a que algunas empresas apuestan por el teletrabajo, el sector tiende a ser conservador, priorizando la presencialidad. Si bien es cierto que durante la pandemia de 2020 trabajar desde casa se volvió la norma, y existía la opinión generalizada de que dicho modelo prevalecería, los datos muestran que no ha sido así. 

Aproximadamente dos de cada tres puestos no requieren una carrera universitaria, lo cual resulta sorprendente dada la naturaleza técnica de la profesión. Es posible que, pese a no indicar explícitamente una carrera como requisito, después sí haya alguna prueba técnica para evaluar y filtrar a candidatos. 

Finalmente, sólo un 11% de las ofertas incluyen un sguro médico. De los 20 países vistos anteriormente, tan solo Estados Unidos carece de un sistema de salud universal. La gran mayoría de la población depende de seguros privados, generalmente vinculados a su empleador. Pero el resto de países tienen algún tipo de sistema público (España, Reino Unido) o, en caso de ser privado, corre a cuenta del ciudadano (Suiza). Es por ello que muy probablemente ese 11% corresponda mayormente a ofertas en Estados Unidos.


