# # TOP 10 barrios con mayor demanda

# En esta primera parte, el análisis consiste en determinar cuáles son los barrios con  mayor demanda como destino. 
# También será importante determinar las compañías de taxis más solicitadas.
# 
# Para ello, es necesario, primero que nada, hacer una exploración de las tablas:
#     
#     - Company -> contiene información sobre el nombre de las empresas de taxis y el número de viajes totales.
#     
#     - Trips -> contiene los barrios donde han terminado los viajes, así como el número de viajes que han terminado en dichos
#     barrios.
#     
# con el fin de verificar la información (que esté completa, no haya valores duplicados y que el tipo de dato sea el adeduado 
# para su manejo) para poder tener una fuente de información confiable.

# In[1]:


import pandas as pd
from matplotlib import pyplot as plt
from scipy.stats import levene, ttest_ind


# In[2]:


company = pd.read_csv('/datasets/project_sql_result_01.csv')
barrios = pd.read_csv('/datasets/project_sql_result_04.csv')



# In[3]:


company.info()


# In[4]:


company.sample(10)


# In[5]:


print('Valores nulos:')
print(company.isnull().sum())
print()
print('Valores duplicados:')
print(company.duplicated().sum())


# In[6]:


sorted(company['company_name'].unique())


# In[32]:


print(company.nlargest(10, 'trips_amount'))


# Observaciones de la tabla Company:
# 
# La tabla Company no tiene valores nulos ni duplicados.
# Consta de dos columnas:
#     
#     company_name -> tipo object
#     
#     trips_amount -> tipo int64
# 
# Ambos títulos cumplen los las condiciones para el nombre.

# In[7]:


barrios.info()


# In[8]:


barrios.sample(10)


# In[9]:


print('Valores nulos:')
print(barrios.isnull().sum())
print()
print('Valores duplicados:')
print(barrios.duplicated().sum())


# In[10]:


sorted(barrios['dropoff_location_name'].unique())


# Observaciones de la tabla Barrios:
# 
# La tabla Barrios no tiene valores nulos ni duplicados.
# Consta de dos columnas:
#     
#     dropoff_location_name -> tipo object 
#     
#     average_trips -> tipo float64
# 
# Ambos títulos cumplen los las condiciones para el nombre.



# ## Identificar los 10 principales barrios en términos de finalización del recorrido

# In[11]:


graphic_barrios = barrios.sort_values(by='average_trips', ascending=False)


# In[12]:


top_10 = graphic_barrios.nlargest(10, 'average_trips')
top_10


# In[13]:


top_10.plot(title='Barrios más solicitados', x='dropoff_location_name', xlabel='Barrio de destino', y='average_trips',
             ylabel='Promedio de viajes', kind = 'bar', rot=45, figsize=[10,10] )
plt.show()


# Los barrios  más solicitados por los usuarios son:
# 
#     Loop
# 
#     River North
# 
#     Streeterville
# 
#     West Loop
# 
#     O'Hare
# 
#     Lake View
# 
#     Grant Park
# 
#     Museum Campus
# 
#     Gold Coast
# 
#     Sheffield & DePaul

# In[14]:


graphic_companies = company.sort_values(by='trips_amount', ascending=False)


# In[15]:


top_10_companies = graphic_companies.nlargest(10, 'trips_amount')
top_10_companies


# In[16]:


top_10_companies.plot(title='Compañías con más viajes', x='company_name', xlabel='Compañía',
                       y='trips_amount', ylabel='Número de viajes', kind = 'bar', rot=90, figsize=[10,10] )
plt.show()


# Las 10 compañías con más viajes solicitados son:
# 
#     Flash Cab
#     
#     Taxi Affiliation Services
#     
#     Medallion Leasing
#     
#     Yellow Cab
#     
#     Taxi Affiliation Service Yellow
#     
#     Chicago Carriage Cab Corp
#     
#     City Service
#     
#     Sun Taxi
#     
#     Star North Management LLC
#     
#     Blue Ribbon Taxi Association Inc.



# # Prueba de hipótesis 

# El objetivo del siguiente análisis, es demostrar la hipótesis planteada.
#     
# Para ello, es necesario, primero que nada, hacer una exploración de la tabla, con el fin de verificar la información (que esté completa, no haya valores duplicados y que el tipo de dato sea el adeduado para su manejo) para poder tener una fuente de información confiable.
# 
# HIPÓTESIS:
# 
# H nula : "La duración promedio de los viajes desde el Loop hasta el Aeropuerto Internacional O'Hare cambia los sábados lluviosos".
# 
# 
# H alternativa : "La duración promedio de los viajes que se realizan los sábados desde Loop hasta el Aeropuerto Internacional O'Hare es la misma durante los días lluviosos y los no lluviosos."

# In[17]:


viajes = pd.read_csv('/datasets/project_sql_result_07.csv')
viajes


# In[18]:


print('Valores nulos:')
print(viajes.isnull().sum())
print()
print('Valores duplicados:')
print(viajes.duplicated().sum())


# In[19]:


viajes.info()


# Convertir los valores de la columna 'start_ts' al tipo datetime para obtener los viajes que se hicieron los sábados:

# In[20]:


viajes['start_ts'] = pd.to_datetime(viajes['start_ts'])


# In[21]:


viajes.info()


# Obtener los viajes que se efectuaron en sábado:

# In[22]:


viajes = viajes[viajes['start_ts'].dt.dayofweek == 5] 


# Dividir el dataframe en dos para obtener los días lluviosos y días buenos:

# In[23]:


bad_days = viajes[viajes['weather_conditions'] == 'Bad']
good_days = viajes[viajes['weather_conditions'] == 'Good']


# Prueba de Levene para evaluar la igualdad de varianzas entre ambos dataframes:

# In[25]:


bad_days_avg = bad_days['duration_seconds']
good_days_avg = good_days['duration_seconds']

statistic_levene, p_value_levene = levene(bad_days_avg, good_days_avg)

print("Estadístico de la prueba de Levene:", statistic_levene)
print("Valor p:", p_value_levene)

alfa = 0.05
if p_value_levene < alfa:
    print("Hay evidencia para rechazar la hipótesis nula de igualdad de varianzas.")
else:
    print("No hay suficiente evidencia para rechazar la hipótesis nula de igualdad de varianzas.")


# Realizar la prueba t de Student para comparar las medias

# In[35]:


statistic_ttest, p_value_ttest = ttest_ind(bad_days_avg, good_days_avg, equal_var=(p_value_levene > 0.05))

alpha = 0.05

print("Duración promedio de los viajes en días lluviosos:", bad_days_avg.mean())
print()
print("Duración promedio de los viajes en días no lluviosos:", good_days_avg.mean())
print()
print("Estadístico del test t de Student:", statistic_ttest)
print()
print("Valor p del test t de Student:", p_value_ttest)
print()
if p_value_ttest < alpha:
    print("No se rechaza la hipótesis nula. Hay una diferencia significativa en la duración promedio de los viajes entre días lluviosos y no lluviosos.")
else:
    print("Se rechaza la hipótesis nula. No hay suficiente evidencia para afirmar que hay una diferencia significativa en la duración promedio de los viajes entre días lluviosos y no lluviosos.")


# CONCLUSIÓN GENERAL
# 
# 
# Identificar los 10 principales barrios en términos de finalización del recorrido
# 
# Los resultados del análisis de la primera sección demuestran que estas son las compañías con mayor demanda y estos los barrios
# con mayor demanda como destino, en la ciudad de Chicago:
# 
# 
# Las compañías más solicitadas son:						Los barrios con mayor demanda como destino son:
# 
# Flash Cab        					19558				Loop				10727.466667
# 
# Taxi Affiliation Services        	11422				River North			9523.666667
# 
# Medallion Leasing         			10367				Streeterville		6664.666667
# 
# Yellow Cab          				9888				West Loop			5163.666667
# 
# Taxi Affiliation Service Yellow     9299				O'Hare				2546.900000
# 
# Chicago Carriage Cab Corp           9181				Lake View			2420.966667
# 
# City Service          				8448				Grant Park			2068.533333
# 
# Sun Taxi          					7701				Museum Campus		1510.000000
# 
# Star North Management LLC           7455				Gold Coast			1364.233333
# 
# Blue Ribbon Taxi Association Inc.   5953				Sheffield & DePaul	1259.766667	
# 
# 
# 
# 
Conclusión de la prueba de hipótesis:
# 
# 
# De acuerdo al análisis, hay diferencia en la duración de los viajes realizados entre los sábados lluviosos y los no lluviosos,
# en promedio 40 minutos los días lluviosos contra 33 minutos los días no lluviosos.


