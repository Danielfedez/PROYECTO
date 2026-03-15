# 5. ¿Cuál es la habilidad más óptima para los analistas de datos? 

Para acceder al documento con el código, clica [aquí](Pregunta_5_code.ipynb)

En esta sección estudiamos qué habilidades son las más óptimas para un analista de datos, es decir, aquellas que presentan alta demanda y alta remuneración para este rol.

Queremos crear un DataFrame que nos indique, para cada habilidad, qué sueldo medio ofrece y en qué proporción respecto al total de puestos se demanda. De esta forma podemos filtrar por las habilidades más populares. 

Eliminamos las filas que no contienen datos de salario y filtramos por *Data Analyst*. Para poder estudiar las habilidades usamos `explode` sobre la columna `job_skills`. 

```
df = df.dropna(subset="salary_year_avg")

df = df[df["job_title_short"] == "Data Analyst"]

df_exploded = df.explode("job_skills")
```
Creamos un nuevo DataFrame, `df_percentage`, con el recuento de cada habilidad y su sueldo medio. Escogemos solamente aquellas habilidades que aparezcan en, al menos, el 5% de las ofertas. 

```
df_skills = df_exploded.groupby('job_skills')
['salary_year_avg'].agg(['count', 'median']).sort_values(by='count', ascending=False)

df_skills = df_skills.rename(columns={'count': 'skill_count', 'median': 'median_salary'})

total_skills = len(df["job_skills"])-1

df_skills["skill_percent"] = df_skills["skill_count"]*100/total_skills

df_percentage = df_skills[df_skills["skill_percent"] >= 5]``
```

| job_skills   |   skill_count |   median_salary |   skill_percent |
|:-------------|--------------:|----------------:|----------------:|
| sql          |          3079 |         92500   |        56.4954  |
| excel        |          2135 |         84479   |        39.1743  |
| python       |          1838 |         98500   |        33.7248  |
| tableau      |          1657 |         95000   |        30.4037  |
| r            |          1073 |         92527.5 |        19.6881  |
| power bi     |          1042 |         90000   |        19.1193  |
| sas          |          1006 |         90000   |        18.4587  |
| word         |           523 |         80000   |         9.59633 |
| powerpoint   |           518 |         85000   |         9.50459 |
| sql server   |           336 |         92150   |         6.16514 |
| oracle       |           332 |         95000   |         6.09174 |
| azure        |           312 |        100000   |         5.72477 |
| aws          |           294 |        100500   |         5.3945  |
| go           |           288 |         90000   |         5.2844  |

Una buena forma de visualizar estos datos es con un diagrama de dispersión, donde veamos la correlación entre abundancia y sueldo para cada habilidad. Hacemos algunos ajustes para que la comprensión del diagrama sea más clara. Importamos `adjustText` para que las etiquetas de las habilidades no se solapen.

```
plt.scatter(df_percentage['skill_percent'], df_percentage['median_salary'])

plt.xlabel('Porcentaje de puestos de analista de datos')

plt.ylabel('Salario Medio ($USD)')  

plt.title('Habilidades más óptimas para un analista de datos')


ax = plt.gca()

ax.yaxis.set_major_formatter(plt.FuncFormatter(lambda y, pos: f'${int(y/1000)}K'))  


texts = []
for i, txt in enumerate(df_percentage.index):
    texts.append(plt.text(df_percentage['skill_percent'].iloc[i], df_percentage['median_salary'].iloc[i], " " + txt))

from adjustText import adjust_text

adjust_text(texts, arrowprops=dict(arrowstyle='->', color='gray'))

plt.show()
```

![optimal_skill_scatter](Imágenes/optimal_skill_scatter.png)

En el extremo derecho, SQL se consolida como la habilidad más solicitada (cerca del 60% de los puestos) pero con un salario medio moderado de $92K, mientras que herramientas de nicho como AWS y Azure ofrecen los salarios más altos, superando los $100K, a pesar de su baja frecuencia en las ofertas. El análisis destaca a Python como la habilidad más equilibrada, combinando una demanda sólida superior al 30% con un atractivo salario de $98K, contrastando drásticamente con herramientas administrativas poco técnicas como Word o PowerPoint, que presentan tanto la menor demanda como la remuneración más baja del ecosistema. 
