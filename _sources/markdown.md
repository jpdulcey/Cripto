```python
import numpy as np
import pandas as pd
from bokeh.io import curdoc, output_notebook
from bokeh.models import HoverTool, ColumnDataSource, CategoricalColorMapper, Slider, CustomJS
from bokeh.palettes import Spectral6
from bokeh.layouts import widgetbox, row
from ipywidgets import interact
from bokeh.io import push_notebook, show, output_notebook
from bokeh.layouts import column
from bokeh.plotting import Figure, output_file, show
import plotly.express as px
import seaborn as sns
import matplotlib.pyplot as plt
import altair as alt
import warnings
from plotly.graph_objects import Layout
import plotly.graph_objects as go
import plotly.figure_factory as ff
warnings.filterwarnings('ignore')
import datetime
from sklearn.preprocessing import MinMaxScaler
from sqlalchemy import create_engine
import psycopg2
```

```python
data=pd.read_csv('https://raw.githubusercontent.com/jpdulcey/Trabajo_Final/main/Consolidado.csv', )
round(data, 2)
engine = create_engine('postgresql://vnnulziinextdf:dfcf04514600a32bd3d989cc5557e3adfa20feccc793e3444896b2ea98e4ff53@ec2-35-168-80-116.compute-1.amazonaws.com:5432/debjgeubaklpbb')
data.to_sql('bitcoin_commodities', engine, if_exists = 'replace', index=False, method='multi')
connection = psycopg2.connect(user="vnnulziinextdf",
                                  password="dfcf04514600a32bd3d989cc5557e3adfa20feccc793e3444896b2ea98e4ff53",
                                  host="ec2-35-168-80-116.compute-1.amazonaws.com",
                                  port="5432",
                                  database="debjgeubaklpbb")
cursor = connection.cursor()
cursor.execute("SELECT * from bitcoin_commodities;")
close = cursor.fetchall()
BTC_sql = pd.DataFrame(close)
display(BTC_sql.head())

cursor.execute("SELECT column_name FROM information_schema.columns WHERE table_name='bitcoin_commodities';")
BTC_names = cursor.fetchall()
display(BTC_names)

print("Operation done successfully")
connection.close()
names_list = [name[0] for name in BTC_names]
BTC_sql.columns = names_list 
BTC_sql = BTC_sql.sort_values(by = "Date")
```

```python
BTC_sql=BTC_sql.fillna(method='ffill')
BTC_sql.info()
```

```python
BTC_sql.Close_Bit
```

```python
BTC_sql["Date"]=pd.to_datetime(BTC_sql["Date"])
BTC_sql.head()
```

```python
n_BTC = len(BTC_sql.Close_Bit); n_test = 28
train_size = n_BTC - n_test

train = BTC_sql.Close_Bit.iloc[:train_size]
test_1w = BTC_sql.Close_Bit.iloc[train_size:train_size + n_test] 
dates_1w = BTC_sql.Date.iloc[train_size:train_size + n_test] 
print("train:", train.shape)
print("test_1w:", test_1w.shape)
test_1w
```

```python
from statsmodels.tsa.arima.model import ARIMA
import warnings
warnings.filterwarnings("ignore")
```

```python
best_aic = np.inf
best_bic = np.inf

best_order = None
best_mdl = None

pq_rng = range(5)
d_rng  = range(3)

for i in pq_rng: 
    for d in d_rng: 
        for j in pq_rng:
            try:
                tmp_mdl = ARIMA(train, order=(i,d,j)).fit() 
                tmp_aic = tmp_mdl.aic                
                if tmp_aic < best_aic: 
                    best_aic = tmp_aic
                    best_order = (i, d, j)
                    best_mdl = tmp_mdl
            except: continue
print('aic: {:6.5f} | order: {}'.format(best_aic, best_order))
```

```python
from statsmodels.tsa.arima.model import ARIMA
import warnings
warnings.filterwarnings("ignore")
model = ARIMA(train, order=(best_order), enforce_stationarity=False).fit(method='innovations_mle')
```

### Error del modelo de predicción


El tamaño del error de proyección es bajo, (5%), entonces el modelo de predicción es muy bueno.
