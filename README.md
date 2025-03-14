import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns

from google.colab import drive
drive.mount('/content/drive')

df = pd.read_csv('/content/drive/My Drive/Estadistica/England 2 CSV.csv')
df

df.info()

df = df.dropna()
df.info()

df['League'].unique()

df['Date'].unique()

df['Display_Order'].unique()

top_local = df.groupby("HomeTeam")["FTH Goals"].sum().sort_values(ascending=False).head(5)
print("Top 5 equipos con más goles de local:")
print(top_local)

top_visitante = df.groupby("AwayTeam")["FTA Goals"].sum().sort_values(ascending=False).head(5)
print("Top 5 equipos con más goles de visitante:")
print(top_visitante)

plt.figure(figsize=(12,6))
df.groupby("HomeTeam")[["FTH Goals", "FTA Goals"]].sum().sum(axis=1).sort_values(ascending=False).head(10).plot(kind="bar", color="orange")
plt.title("Top 10 Equipos con más Goles en la Temporada")
plt.xlabel("Equipo")
plt.ylabel("Total de Goles")
plt.xticks(rotation=90)
plt.show()

# Filtrar partidos que terminaron en empate
empates_local = df[df["FT Result"] == "D"].groupby("HomeTeam")["FT Result"].count()
empates_visitante = df[df["FT Result"] == "D"].groupby("AwayTeam")["FT Result"].count()

# Unir los empates de local y visitante
empates_por_equipo = empates_local.add(empates_visitante, fill_value=0)

# Ordenar los equipos con más empates
empates_por_equipo = empates_por_equipo.sort_values(ascending=False)

# Mostrar los 10 equipos con más empates
print("Top 10 equipos con más empates en la temporada:")
print(empates_por_equipo.head(10))

plt.figure(figsize=(12,6))
sns.barplot(
    y=empates_por_equipo.index[:10],
    x=empates_por_equipo[:10],
    palette="Greens_r"
)
plt.title("Top 10 Equipos con más Empates en la Temporada")
plt.xlabel("Total de Empates")
plt.ylabel("Equipo")
plt.show()

plt.figure(figsize=(6,4))
sns.violinplot(x=df["FT Result"], inner="point", palette="muted")
plt.title("Distribución de Resultados (Local, Empate, Visitante)")
plt.xlabel("Resultado Final")
plt.show()

plt.figure(figsize=(6,6))
df["FT Result"].value_counts().plot(kind="pie", autopct="%1.1f%%", colors=["#ff9999","#66b3ff","#99ff99"])
plt.title("Distribución de Resultados (Local, Empate, Visitante)")
plt.ylabel("")  # Ocultamos la etiqueta del eje Y
plt.show()

# Convertir Date a formato datetime si no se ha hecho
df["Date"] = pd.to_datetime(df["Date"], dayfirst=True)

# Agrupar por fecha y calcular el total de goles en cada jornada
goles_por_fecha = df.groupby("Date")[["FTH Goals", "FTA Goals"]].sum()
goles_por_fecha["Total Goles"] = goles_por_fecha["FTH Goals"] + goles_por_fecha["FTA Goals"]

# Encontrar la fecha con más goles
fecha_max_goles = goles_por_fecha["Total Goles"].idxmax()
max_goles = goles_por_fecha["Total Goles"].max()

# Encontrar la fecha con menos goles
fecha_min_goles = goles_por_fecha["Total Goles"].idxmin()
min_goles = goles_por_fecha["Total Goles"].min()

print(f"📅 Fecha con MÁS goles: {fecha_max_goles} ({max_goles} goles)")
print(f"📅 Fecha con MENOS goles: {fecha_min_goles} ({min_goles} goles)")

# Aplicar estilo de gráficos
plt.style.use("seaborn-v0_8-darkgrid")

# Suavizar la línea con un promedio móvil de 7 días
goles_por_fecha["Promedio Móvil"] = goles_por_fecha["Total Goles"].rolling(window=7).mean()

# Crear la figura
plt.figure(figsize=(12, 6))

# Graficar la evolución de los goles con una línea suavizada
plt.plot(goles_por_fecha.index, goles_por_fecha["Promedio Móvil"], linestyle='-', color='royalblue', linewidth=2, label="Promedio Móvil (7 días)")

# Destacar el año más cercano a 49 goles
anio_cercano_49 = goles_por_fecha.iloc[(goles_por_fecha["Total Goles"] - 49).abs().argmin()]
plt.scatter(anio_cercano_49.name, anio_cercano_49["Total Goles"], color='gold', s=150, edgecolors="black", zorder=3)
plt.annotate(f"Año cercano a 49 goles\n({anio_cercano_49.name.year})", xy=(anio_cercano_49.name, anio_cercano_49["Total Goles"]),
             xytext=(anio_cercano_49.name, anio_cercano_49["Total Goles"] + 3),
             ha='center', fontsize=10, color='gold', fontweight='bold', bbox=dict(facecolor="black", alpha=0.1))

# Destacar el año con menos goles
anio_min_goles = goles_por_fecha["Total Goles"].idxmin()
min_goles = goles_por_fecha.loc[anio_min_goles, "Total Goles"]
plt.scatter(anio_min_goles, min_goles, color='red', s=150, edgecolors="black", zorder=3)
plt.annotate(f"Año con menos goles\n({anio_min_goles.year})", xy=(anio_min_goles, min_goles),
             xytext=(anio_min_goles, min_goles + 3),
             ha='center', fontsize=10, color='red', fontweight='bold', bbox=dict(facecolor="black", alpha=0.1))

# Personalizar la gráfica
plt.title("Evolución de Goles por Fecha en la English Championship", fontsize=14, fontweight="bold")
plt.xlabel("Fecha", fontsize=12)
plt.ylabel("Total de Goles", fontsize=12)
plt.xticks(rotation=45)
plt.legend()
plt.grid(True, linestyle="--", alpha=0.5)
plt.tight_layout()  # Optimiza el diseño

# Mostrar la gráfica
plt.show()

# Contar la cantidad de partidos dirigidos por cada árbitro
arbitros_mas_partidos = df["Referee"].value_counts().head(10)  # Top 10 árbitros con más partidos

# Crear la gráfica
plt.figure(figsize=(12, 6))
arbitros_mas_partidos.plot(kind="bar", color="royalblue", alpha=0.7)

# Personalizar la gráfica
plt.title("Árbitros con Más Partidos Dirigidos", fontsize=14, fontweight="bold")
plt.xlabel("Árbitro", fontsize=12)
plt.ylabel("Cantidad de Partidos", fontsize=12)
plt.xticks(rotation=45, ha="right")
plt.grid(axis="y", linestyle="--", alpha=0.5)
plt.tight_layout()

# Mostrar la gráfica
plt.show()

# Asegurar que la columna Season es de tipo string (por si está en otro formato)
df["Season"] = df["Season"].astype(str)

# Contar la cantidad de partidos dirigidos por cada árbitro en cada temporada
arbitros_por_temporada = df.groupby(["Season", "Referee"]).size().unstack(fill_value=0)

# Seleccionar los árbitros con más participación total
top_arbitros = arbitros_por_temporada.sum().sort_values(ascending=False).head(5).index

# Filtrar los datos solo para esos árbitros
arbitros_filtrados = arbitros_por_temporada[top_arbitros]

# Graficar la evolución
plt.figure(figsize=(12, 6))
for arbitro in arbitros_filtrados.columns:
    plt.plot(arbitros_filtrados.index, arbitros_filtrados[arbitro], marker="o", linestyle="-", label=arbitro)

# Personalizar la gráfica
plt.title("Participación de los Árbitros por Temporada", fontsize=14, fontweight="bold")
plt.xlabel("Temporada", fontsize=12)
plt.ylabel("Partidos Dirigidos", fontsize=12)
plt.xticks(rotation=45)
plt.legend(title="Árbitro", bbox_to_anchor=(1.05, 1), loc="upper left")  # Mueve la leyenda fuera del gráfico
plt.grid(True, linestyle="--", alpha=0.5)
plt.tight_layout()

# Mostrar la gráfica
plt.show()

# Asegurar que la columna "Season" es tipo string
df["Season"] = df["Season"].astype(str)

# Contar la cantidad de cada resultado por temporada
resultados_por_temporada = df.groupby("Season")["FT Result"].value_counts().unstack(fill_value=0)

# Graficar la evolución
plt.figure(figsize=(12, 6))
for resultado in resultados_por_temporada.columns:
    plt.plot(resultados_por_temporada.index, resultados_por_temporada[resultado], marker="o", linestyle="-", label=resultado)

# Personalizar
plt.title("Evolución de Resultados Finales por Temporada", fontsize=14, fontweight="bold")
plt.xlabel("Temporada", fontsize=12)
plt.ylabel("Cantidad de Partidos", fontsize=12)
plt.xticks(rotation=45)
plt.legend(title="Resultado", loc="upper left")
plt.grid(True, linestyle="--", alpha=0.5)
plt.tight_layout()

# Mostrar
plt.show()

# Asegurar que la columna "Season" es string
df["Season"] = df["Season"].astype(str)

# Sumar faltas por temporada
faltas_por_temporada = df.groupby("Season")[["H Fouls", "A Fouls"]].sum()

# Graficar
plt.figure(figsize=(12, 6))
plt.plot(faltas_por_temporada.index, faltas_por_temporada["H Fouls"], marker="o", linestyle="-", label="Faltas Locales", color="royalblue")
plt.plot(faltas_por_temporada.index, faltas_por_temporada["A Fouls"], marker="o", linestyle="-", label="Faltas Visitantes", color="red")

# Personalizar
plt.title("Evolución de Faltas por Temporada", fontsize=14, fontweight="bold")
plt.xlabel("Temporada", fontsize=12)
plt.ylabel("Total de Faltas", fontsize=12)
plt.xticks(rotation=45)
plt.legend()
plt.grid(True, linestyle="--", alpha=0.5)
plt.tight_layout()

# Mostrar
plt.show()

# Sumar las faltas cometidas por cada equipo (local y visitante)
faltas_locales = df.groupby("HomeTeam")["H Fouls"].sum()
faltas_visitantes = df.groupby("AwayTeam")["A Fouls"].sum()

# Combinar ambas columnas para obtener el total de faltas por equipo
faltas_por_equipo = faltas_locales.add(faltas_visitantes, fill_value=0)

# Convertir a valores numéricos
faltas_totales = faltas_por_equipo.values.astype(int)

# Graficar el histograma
plt.figure(figsize=(10, 6))
plt.hist(faltas_totales, bins=10, color="purple", alpha=0.7, edgecolor="black")

# Personalizar
plt.title("Distribución de Faltas Cometidas por los Equipos", fontsize=14, fontweight="bold")
plt.xlabel("Cantidad de Faltas", fontsize=12)
plt.ylabel("Número de Equipos", fontsize=12)
plt.grid(axis="y", linestyle="--", alpha=0.5)

# Mostrar
plt.show()

df["Display_Order"].dtype

df["Display_Order"].unique()

df["Display_Order"].value_counts()

plt.figure(figsize=(10, 6))
plt.hist(df["Display_Order"], bins=20, color="royalblue", alpha=0.7, edgecolor="black")

plt.title("Distribución de Display_Order", fontsize=14, fontweight="bold")
plt.xlabel("Display_Order", fontsize=12)
plt.ylabel("Frecuencia", fontsize=12)
plt.grid(axis="y", linestyle="--", alpha=0.5)

plt.show()

plt.figure(figsize=(8, 5))
sns.boxplot(x=df["Display_Order"], color="orange")

plt.title("Boxplot de Display_Order", fontsize=14, fontweight="bold")
plt.xlabel("Display_Order", fontsize=12)

plt.show()

# Asegurar que la columna 'Date' sea de tipo fecha
df["Date"] = pd.to_datetime(df["Date"])

# Crear la figura
plt.figure(figsize=(12, 6))

# Graficar la evolución de Display_Order con puntos en lugar de línea
plt.scatter(df["Date"], df["Display_Order"], color="green", alpha=0.7)

# Personalizar la gráfica
plt.title("Evolución de Display_Order en el Tiempo", fontsize=14, fontweight="bold")
plt.xlabel("Fecha", fontsize=12)
plt.ylabel("Display_Order", fontsize=12)

# Ajustar etiquetas del eje X para que no se superpongan
plt.xticks(rotation=45)
plt.xticks(df["Date"][::len(df)//10])  # Mostrar solo algunas fechas

# Ajustar la cuadrícula para que no sature
plt.grid(True, linestyle="--", alpha=0.3)

# Mostrar la gráfica
plt.show()

# Sumar faltas cometidas por cada equipo como local y visitante
faltas_locales = df.groupby("HomeTeam")["H Fouls"].sum()
faltas_visitantes = df.groupby("AwayTeam")["A Fouls"].sum()

# Sumar ambas para obtener el total de faltas por equipo
faltas_totales = faltas_locales.add(faltas_visitantes, fill_value=0)

# Encontrar el equipo con más faltas
equipo_mas_faltas = faltas_totales.idxmax()
max_faltas = faltas_totales.max()

print(f"El equipo con más faltas es {equipo_mas_faltas} con un total de {max_faltas} faltas.")

plt.figure(figsize=(12, 6))
plt.hist(df["H Fouls"], bins=15, alpha=0.6, color="blue", edgecolor="black", label="Faltas Local (H Fouls)")
plt.hist(df["A Fouls"], bins=15, alpha=0.6, color="red", edgecolor="black", label="Faltas Visitante (A Fouls)")

plt.title("Distribución de Faltas en Partidos", fontsize=14, fontweight="bold")
plt.xlabel("Número de Faltas", fontsize=12)
plt.ylabel("Frecuencia", fontsize=12)
plt.legend()
plt.grid(axis="y", linestyle="--", alpha=0.5)

plt.show()

df["Season"] = df["Date"].dt.year  # Convertir fechas a años si no tienes una columna de temporada

faltas_por_temporada = df.groupby("Season")[["H Fouls", "A Fouls"]].mean()

plt.figure(figsize=(12, 6))
plt.plot(faltas_por_temporada.index, faltas_por_temporada["H Fouls"], marker="o", linestyle="-", color="blue", label="Faltas Locales")
plt.plot(faltas_por_temporada.index, faltas_por_temporada["A Fouls"], marker="o", linestyle="-", color="red", label="Faltas Visitantes")

plt.title("Promedio de Faltas por Temporada", fontsize=14, fontweight="bold")
plt.xlabel("Temporada", fontsize=12)
plt.ylabel("Promedio de Faltas", fontsize=12)
plt.legend()
plt.grid(True, linestyle="--", alpha=0.5)

plt.show()

# Crear la figura
plt.figure(figsize=(8, 6))

# Graficar el boxplot
sns.boxplot(data=df[["H Fouls", "A Fouls"]], palette=["blue", "red"])

# Personalizar la gráfica
plt.title("Distribución de Faltas Locales y Visitantes", fontsize=14, fontweight="bold")
plt.ylabel("Número de Faltas", fontsize=12)
plt.xticks([0, 1], ["Faltas Locales", "Faltas Visitantes"])  # Etiquetas personalizadas en el eje X

# Mostrar la gráfica
plt.show()

# Asegurar que la columna 'Date' sea tipo fecha
df["Date"] = pd.to_datetime(df["Date"])

# Extraer la temporada (año) desde la fecha
df["Season"] = df["Date"].dt.year

# Contar la cantidad de partidos dirigidos por cada árbitro en cada temporada
arbitros_por_temporada = df.groupby(["Season", "Referee"]).size().reset_index(name="Partidos")

# Obtener los árbitros con más partidos en cada temporada
top_arbitros = arbitros_por_temporada.loc[arbitros_por_temporada.groupby("Season")["Partidos"].idxmax()]

# Mostrar la tabla
print(top_arbitros)

# Crear una nueva columna con el total de corners por partido
df["Total Corners"] = df["H Corners"] + df["A Corners"]

# Graficar histograma
plt.figure(figsize=(10, 5))
plt.hist(df["Total Corners"], bins=15, color="blue", alpha=0.7, edgecolor="black")

# Personalizar
plt.title("Distribución de Corners por Partido", fontsize=14, fontweight="bold")
plt.xlabel("Cantidad de Corners en un Partido", fontsize=12)
plt.ylabel("Frecuencia", fontsize=12)
plt.grid(axis="y", linestyle="--", alpha=0.5)

# Mostrar
plt.show()

plt.figure(figsize=(8, 6))
sns.scatterplot(x=df["H Corners"], y=df["A Corners"], alpha=0.7, color="purple")

# Personalizar
plt.title("Relación entre Corners Locales y Visitantes", fontsize=14, fontweight="bold")
plt.xlabel("Corners del Equipo Local", fontsize=12)
plt.ylabel("Corners del Equipo Visitante", fontsize=12)
plt.grid(True, linestyle="--", alpha=0.5)

# Mostrar
plt.show()


plt.figure(figsize=(8, 6))
sns.heatmap(df[["H Corners", "A Corners", "H Fouls", "A Fouls"]].corr(), annot=True, cmap="coolwarm", fmt=".2f")

# Personalizar
plt.title("Correlación entre Corners y Faltas", fontsize=14, fontweight="bold")

# Mostrar
plt.show()
#

import matplotlib.pyplot as plt
import pandas as pd

# Asegurar que la columna "Date" es de tipo datetime
df["Date"] = pd.to_datetime(df["Date"], dayfirst=True)

# Agrupar por semana o mes para suavizar los datos
corners_por_fecha = df.resample("W", on="Date")[["H Corners", "A Corners"]].mean()

# Crear la figura
plt.figure(figsize=(12, 6))

# Graficar con menos puntos (una media semanal)
plt.plot(corners_por_fecha.index, corners_por_fecha["H Corners"], label="Corners Local", marker="o", linestyle="-", alpha=0.7, color="blue")
plt.plot(corners_por_fecha.index, corners_por_fecha["A Corners"], label="Corners Visitante", marker="s", linestyle="--", alpha=0.7, color="red")

# Personalizar
plt.title("Evolución del Promedio de Corners (Suavizado por Semana)", fontsize=14, fontweight="bold")
plt.xlabel("Fecha", fontsize=12)
plt.ylabel("Promedio de Corners", fontsize=12)
plt.xticks(rotation=45)
plt.legend()
plt.grid(True, linestyle="--", alpha=0.5)
plt.tight_layout()

# Mostrar
plt.show()

# Asegurar que la columna "Date" sea de tipo datetime
df["Date"] = pd.to_datetime(df["Date"], dayfirst=True)

# Agrupar por semana para suavizar los datos
tarjetas_por_fecha = df.resample("W", on="Date")[["H Yellow", "A Yellow", "H Red", "A Red"]].sum()

# Crear la figura
plt.figure(figsize=(12, 6))

# Graficar la evolución
plt.plot(tarjetas_por_fecha.index, tarjetas_por_fecha["H Yellow"] + tarjetas_por_fecha["A Yellow"],
         label="Tarjetas Amarillas", marker="o", linestyle="-", color="gold", alpha=0.7)
plt.plot(tarjetas_por_fecha.index, tarjetas_por_fecha["H Red"] + tarjetas_por_fecha["A Red"],
         label="Tarjetas Rojas", marker="s", linestyle="--", color="red", alpha=0.7)

# Personalizar
plt.title("Evolución de Tarjetas Amarillas y Rojas", fontsize=14, fontweight="bold")
plt.xlabel("Fecha", fontsize=12)
plt.ylabel("Número de Tarjetas", fontsize=12)
plt.xticks(rotation=45)
plt.legend()
plt.grid(True, linestyle="--", alpha=0.5)
plt.tight_layout()

# Mostrar
plt.show()

# Asegurar que la columna "Season" es de tipo string
df["Season"] = df["Season"].astype(str)

# Sumar los goles por temporada
goles_por_temporada = df.groupby("Season")[["FTH Goals", "FTA Goals"]].sum()

# Calcular el total de goles
goles_por_temporada["Total Goles"] = goles_por_temporada["FTH Goals"] + goles_por_temporada["FTA Goals"]

# Graficar
plt.figure(figsize=(10, 6))
goles_por_temporada["Total Goles"].plot(kind="bar", color="royalblue", alpha=0.7)

# Personalizar
plt.title("Total de Goles por Temporada", fontsize=14, fontweight="bold")
plt.xlabel("Temporada", fontsize=12)
plt.ylabel("Goles Totales", fontsize=12)
plt.xticks(rotation=45)
plt.grid(axis="y", linestyle="--", alpha=0.5)
plt.tight_layout()

# Mostrar
plt.show()

# Asegurar que "Season" sea tipo string
df["Season"] = df["Season"].astype(str)

# Sumar los goles por temporada
goles_por_temporada = df.groupby("Season")[["FTH Goals", "FTA Goals"]].sum()

# Calcular total de goles por temporada
goles_por_temporada["Total Goles"] = goles_por_temporada["FTH Goals"] + goles_por_temporada["FTA Goals"]

# Graficar la torta
plt.figure(figsize=(8, 8))
plt.pie(
    goles_por_temporada["Total Goles"],
    labels=goles_por_temporada.index,
    autopct="%1.1f%%",
    colors=plt.cm.Paired.colors,
    startangle=140,
    wedgeprops={"edgecolor": "black"}
)

# Personalizar
plt.title("Distribución de Goles por Temporada", fontsize=14, fontweight="bold")
plt.tight_layout()

# Mostrar
plt.show()

# Asegurar que la columna "Season" sea string para evitar problemas en el eje X
df["Season"] = df["Season"].astype(str)

# Contar la cantidad de partidos arbitrados por cada referee en cada temporada
participacion_arbitros = df.groupby(["Season", "Referee"]).size().reset_index(name="Partidos")

# Seleccionar los árbitros con más participación total
top_arbitros = participacion_arbitros.groupby("Referee")["Partidos"].sum().sort_values(ascending=False).head(10).index

# Filtrar solo los árbitros más participativos
participacion_top = participacion_arbitros[participacion_arbitros["Referee"].isin(top_arbitros)].dropna()

# Crear la figura
plt.figure(figsize=(12, 6))

# Graficar evolución de los árbitros más participativos
for referee in top_arbitros:
    data_arbitro = participacion_top[participacion_top["Referee"] == referee]
    plt.plot(data_arbitro["Season"], data_arbitro["Partidos"], marker="o", linestyle="-", label=referee)

# Personalizar gráfico
plt.title("Evolución de la Participación de los Árbitros por Temporada", fontsize=14, fontweight="bold")
plt.xlabel("Temporada", fontsize=12)
plt.ylabel("Cantidad de Partidos Arbitrados", fontsize=12)
plt.xticks(rotation=45)
plt.legend(title="Árbitros", bbox_to_anchor=(1.05, 1), loc="upper left")
plt.grid(True, linestyle="--", alpha=0.5)
plt.tight_layout()

# Mostrar la gráfica
plt.show()

# Agrupar por temporada y sumar los goles
goles_por_temporada = df.groupby("Season")[["FTH Goals", "FTA Goals"]].sum()
goles_por_temporada["Total Goles"] = goles_por_temporada["FTH Goals"] + goles_por_temporada["FTA Goals"]

# Graficar
plt.figure(figsize=(12, 6))
sns.barplot(data=goles_por_temporada.reset_index(), x="Season", y="Total Goles", palette="Blues_r")

# Personalización
plt.title("Total de Goles por Temporada", fontsize=14, fontweight="bold")
plt.xlabel("Temporada", fontsize=12)
plt.ylabel("Goles Totales", fontsize=12)
plt.xticks(rotation=45)
plt.grid(axis="y", linestyle="--", alpha=0.5)
plt.show()

plt.figure(figsize=(10, 6))

# Gráfico KDE para comparar SOT de locales y visitantes
sns.kdeplot(df["H SOT"], color="blue", label="Disparos Local (H SOT)", fill=True, alpha=0.3)
sns.kdeplot(df["A SOT"], color="red", label="Disparos Visitante (A SOT)", fill=True, alpha=0.3)

plt.title("Distribución de Disparos a Puerta", fontsize=14, fontweight="bold")
plt.xlabel("Cantidad de Disparos a Puerta", fontsize=12)
plt.ylabel("Densidad", fontsize=12)
plt.legend()
plt.grid(True, linestyle="--", alpha=0.5)
plt.show()

plt.figure(figsize=(10, 6))

sns.histplot(df["H SOT"], bins=15, kde=True, color="blue", label="Disparos Local (H SOT)", alpha=0.6)
sns.histplot(df["A SOT"], bins=15, kde=True, color="red", label="Disparos Visitante (A SOT)", alpha=0.6)

plt.title("Comparación de Disparos a Puerta entre Locales y Visitantes", fontsize=14, fontweight="bold")
plt.xlabel("Cantidad de Disparos a Puerta", fontsize=12)
plt.ylabel("Frecuencia", fontsize=12)
plt.legend()
plt.grid(axis="y", linestyle="--", alpha=0.5)

plt.show()

plt.figure(figsize=(8, 6))

sns.boxplot(data=df[["H SOT", "A SOT"]], palette=["blue", "red"])

plt.title("Distribución de Disparos a Puerta (SOT) de Locales y Visitantes", fontsize=14, fontweight="bold")
plt.ylabel("Cantidad de Disparos a Puerta", fontsize=12)
plt.xticks([0, 1], ["H SOT (Local)", "A SOT (Visitante)"])

plt.grid(axis="y", linestyle="--", alpha=0.5)
plt.show()

plt.figure(figsize=(8, 6))

sns.violinplot(data=df[["H SOT", "A SOT"]], palette=["blue", "red"])

plt.title("Distribución de Disparos a Puerta (SOT) de Locales y Visitantes", fontsize=14, fontweight="bold")
plt.ylabel("Cantidad de Disparos a Puerta", fontsize=12)
plt.xticks([0, 1], ["H SOT (Local)", "A SOT (Visitante)"])

plt.grid(axis="y", linestyle="--", alpha=0.5)
plt.show()

promedios = df[["H SOT", "A SOT"]].mean()

plt.figure(figsize=(6, 5))
sns.barplot(x=promedios.index, y=promedios.values, palette=["blue", "red"])

plt.title("Promedio de Disparos a Puerta por Equipo", fontsize=14, fontweight="bold")
plt.ylabel("Promedio de Disparos a Puerta", fontsize=12)

plt.grid(axis="y", linestyle="--", alpha=0.5)
plt.show()

plt.figure(figsize=(10, 6))

sns.histplot(df["H Shots"], bins=15, kde=True, color="blue", label="Disparos Local (H Shots)", alpha=0.6)
sns.histplot(df["A Shots"], bins=15, kde=True, color="red", label="Disparos Visitante (A Shots)", alpha=0.6)

plt.title("Comparación de Disparos Totales entre Locales y Visitantes", fontsize=14, fontweight="bold")
plt.xlabel("Cantidad de Disparos", fontsize=12)
plt.ylabel("Frecuencia", fontsize=12)
plt.legend()
plt.grid(axis="y", linestyle="--", alpha=0.5)

plt.show()

plt.figure(figsize=(8, 6))

sns.boxplot(data=df[["H Shots", "A Shots"]], palette=["blue", "red"])

plt.title("Distribución de Disparos Totales de Locales y Visitantes", fontsize=14, fontweight="bold")
plt.ylabel("Cantidad de Disparos", fontsize=12)
plt.xticks([0, 1], ["H Shots (Local)", "A Shots (Visitante)"])

plt.grid(axis="y", linestyle="--", alpha=0.5)
plt.show()

plt.figure(figsize=(8, 6))

sns.violinplot(data=df[["H Shots", "A Shots"]], palette=["blue", "red"])

plt.title("Distribución de Disparos Totales de Locales y Visitantes", fontsize=14, fontweight="bold")
plt.ylabel("Cantidad de Disparos", fontsize=12)
plt.xticks([0, 1], ["H Shots (Local)", "A Shots (Visitante)"])

plt.grid(axis="y", linestyle="--", alpha=0.5)
plt.show()

df["Date"] = pd.to_datetime(df["Date"])  # Asegurar que la columna Date es de tipo datetime

plt.figure(figsize=(12, 6))

plt.plot(df["Date"], df["H Shots"], marker="o", linestyle="-", color="blue", alpha=0.6, label="H Shots (Local)")
plt.plot(df["Date"], df["A Shots"], marker="o", linestyle="-", color="red", alpha=0.6, label="A Shots (Visitante)")

plt.title("Evolución de Disparos Totales en el Tiempo", fontsize=14, fontweight="bold")
plt.xlabel("Fecha", fontsize=12)
plt.ylabel("Cantidad de Disparos", fontsize=12)
plt.xticks(rotation=45)
plt.legend()
plt.grid(True, linestyle="--", alpha=0.5)

plt.show()7

promedios = df[["H Shots", "A Shots"]].mean()

plt.figure(figsize=(6, 5))
sns.barplot(x=promedios.index, y=promedios.values, palette=["blue", "red"])

plt.title("Promedio de Disparos Totales por Equipo", fontsize=14, fontweight="bold")
plt.ylabel("Promedio de Disparos", fontsize=12)

plt.grid(axis="y", linestyle="--", alpha=0.5)
plt.show()

