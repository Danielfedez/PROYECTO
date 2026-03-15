# 3. ¿Cuál es la tendencia anual de las 5 habilidades más demandadas para los analistas de datos?

Para acceder al documento con el código, clica [aquí](Pregunta_3_code.ipynb)

Filtramos por *Data Analyst* y usamos `explode` sobre nuestro DataFrame.

```
df = df[df["job_title_short"] == "Data Analyst"]

df_exploded = df.explode("job_skills")
```

Para estudiar la tendencia anual de las cinco habilidades más demandadas es necesario hacer una limpieza técnica en la columna `job_posted_date`. Vemos que, para cada oferta, esta columna nos indica la fecha y la hora en que fue publicada, pero se trata de una cadena de texto. Para que Python la reconozca como una medida de tiempo, usamos `pd.to_datetime()`  sobre dicha columna, con lo cual obtenemos un objeto `datetime64[ns]` que nos facilitará el análisis mes a mes. 


```
df["job_posted_date"] = pd.to_datetime(df["job_posted_date"])
```
Creamos una nueva columna, `job_posted_month`, que nos indica solamente el mes (1-12) de publicación de cada oferta. Definimos `top_5_skills` como una lista con las 5 habilidades con mayor recuento. Agrupamos recuento de habilidades y meses en una `pivot_table`. 

```
df_exploded["job_posted_month"] = df_exploded["job_posted_date"].dt.month

top_5= df_exploded["job_skills"].value_counts().head(5).index.tolist()

tabla = df_exploded.groupby("job_posted_month")["job_skills"].value_counts()

skill_count = tabla.reset_index(name="skill_count")

skill_count= skill_count[skill_count["job_skills"].isin(top_5)]
```

|   job_posted_month |   excel |   power bi |   python |   sql |   tableau |
|-------------------:|--------:|-----------:|---------:|------:|----------:|
|                  1 |    8170 |       4285 |     6606 | 11336 |      5596 |
|                  2 |    5772 |       3307 |     4751 |  7947 |      3936 |
|                  3 |    5675 |       3176 |     4741 |  7868 |      4051 |
|                  4 |    5496 |       3106 |     4557 |  7553 |      3776 |
|                  5 |    4773 |       2695 |     4070 |  6617 |      3245 |
|                  6 |    5724 |       3275 |     4707 |  7584 |      3812 |
|                  7 |    5513 |       3350 |     4831 |  7687 |      3928 |
|                  8 |    6482 |       3859 |     5576 |  8823 |      4533 |
|                  9 |    4886 |       3118 |     4229 |  6829 |      3446 |
|                 10 |    5217 |       3340 |     4693 |  7474 |      3709 |
|                 11 |    4776 |       3012 |     4233 |  6652 |      3284 |
|                 12 |    4376 |       2857 |     4196 |  6058 |      3139 |



Esta tabla se entiende mejor si renombramos los índices de `job_posted_month`, y en vez de ir del 1 al 12, van de Jan a Dec. Importamos `calendar` y creamos un bucle `for` para renombrar los meses.

```
import calendar

pivot.index = [calendar.month_abbr[i] for i in pivot.index]
```

|     |   excel |   power bi |   python |   sql |   tableau |
|:----|--------:|-----------:|---------:|------:|----------:|
| Jan |    8170 |       4285 |     6606 | 11336 |      5596 |
| Feb |    5772 |       3307 |     4751 |  7947 |      3936 |
| Mar |    5675 |       3176 |     4741 |  7868 |      4051 |
| Apr |    5496 |       3106 |     4557 |  7553 |      3776 |
| May |    4773 |       2695 |     4070 |  6617 |      3245 |
| Jun |    5724 |       3275 |     4707 |  7584 |      3812 |
| Jul |    5513 |       3350 |     4831 |  7687 |      3928 |
| Aug |    6482 |       3859 |     5576 |  8823 |      4533 |
| Sep |    4886 |       3118 |     4229 |  6829 |      3446 |
| Oct |    5217 |       3340 |     4693 |  7474 |      3709 |
| Nov |    4776 |       3012 |     4233 |  6652 |      3284 |
| Dec |    4376 |       2857 |     4196 |  6058 |      3139 |


Finalmente representamos gráficamente.

![trending_skills](Imágenes/trending_skills.png)


Esta gráfica de tendencias mensuales revela que, a pesar de las fluctuaciones en el volumen total de vacantes, la jerarquía de las herramientas tecnológicas para los analistas de datos se mantiene extremadamente estable a lo largo del año. SQL lidera de forma indiscutible, consolidándose como el requisito fundamental, seguido por un grupo intermedio muy compacto compuesto por Excel y Python. Es notable observar una estacionalidad compartida entre todas las habilidades: un pico masivo de demanda en enero, seguido de una estabilización con ligeros repuntes en meses como agosto y octubre, lo que sugiere que las empresas ajustan su volumen de contratación de forma uniforme según el ciclo anual del mercado laboral.

Por otro lado, la brecha persistente entre herramientas de visualización como Tableau y Power BI frente al dominio de SQL subraya que la prioridad del mercado sigue siendo la extracción y manipulación de datos sobre su presentación. El hecho de que Python mantenga una tendencia casi idéntica a Excel refleja la naturaleza híbrida del rol, donde conviven habilidades de programación con el análisis tradicional en hojas de cálculo. Para un profesional del sector, esta gráfica es una hoja de ruta de resiliencia: especializarse en las herramientas de la parte superior (SQL/Excel/Python) garantiza alineación con la demanda estructural del mercado, independientemente del mes en que se busque empleo.