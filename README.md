# Production-Hole-Design-B-C-
d = 12.25                                        
de = 1.1                                         
vod = 3947                                  
h= 12                                            
dr = 2.7                                        
rqd = 100                                      
ucs = 1529.58                                  
srb = 1                                         
rd = 0.7                                         

#Dimensiones de la malla:

x_inicio = 20; x_final = 100; y_inicio = 20; y_final = 50

import pandas as pd
import numpy as np
import plotly.express as px
#Cálculos Método Pearse-Borquéz:
ts = (8/100)*ucs                                           
kv = (1.96 - 0.27*np.log(0.7*rqd)).round(1)              
pod = (250*de*(vod**2))*0.000001                           
pod_psi = 145.038* pod                                     
b = int(((0.0254*kv*d*((pod_psi/ts)**0.5))*0.3048).round(0))    
#Cálculos Taco Crítico de Chiappetta:
d_pozo = d*0.0254                                     
lsd = d_pozo*10                                       
ql = 0.507*de*d**2                                    
w = lsd*ql                                           
SD = 1.1                                              
Tcritico = int(round(((SD*(w**0.33))-(0.5*lsd)),0))   
j = 0.2*b                                             
burden = b
espaciamiento= int(b/srb)                             
x_size = x_final - x_inicio
y_size = y_final - y_inicio
x_p = int((x_final - x_inicio)/espaciamiento+1)
y_p = int((y_final - y_inicio)/burden+1)
n_pozos = x_p*y_p
m_per = n_pozos*(h+j)
c_e = (h+j)-Tcritico

espaciamiento_x = []
for x in range(x_inicio, x_final, espaciamiento):
     espaciamiento_x.append(x)  
burden_y = []
for x in range(y_inicio, y_final, burden):
     burden_y.append(x)    
n_burden = len(burden_y)
n_espaciamiento = len(espaciamiento_x)
coordinates = []
for x in range(x_inicio, x_final, espaciamiento):
    for y in range(y_inicio, y_final, burden):
        coordinates.append((x, y))    
df = pd.DataFrame(coordinates)
df.rename(columns={0:'x',1:'y'}, inplace=True)
z = []
for x in np.arange(0.0, (h+j), 0.1):
     z.append(x)
a = z*810
df_z = pd.DataFrame(a)
df_z.rename(columns={0:'z'}, inplace=True)
newdf = pd.DataFrame(np.repeat(df.values,(h+j)*10,axis=0))
newdf.columns = df.columns
df2 = newdf.copy()
df2["z"] = df_z["z"]
# 0 = taco
# 1 = carga columna
#2 = carga de fondo
def f(row):
    if row['z'] >= h-Tcritico:
        val = "Taco"
    elif row['z']<= j:
        val = "Pasadura"
    else:
        val = "Explosivo"
    return val
df2['Carga de pozo'] = df2.apply(f, axis=1)

fig = px.scatter_3d(df2, x="x", y="y",z="z", color="Carga de pozo", 
                    color_discrete_sequence=px.colors.qualitative.Plotly)
fig.update_traces(marker=dict(size=5, symbol = 'circle',
                              line=dict(width=0,color='DarkSlateGrey')), selector=dict(mode='markers'))
fig.update_layout(scene = dict(
                    xaxis = dict(
                         backgroundcolor="rgb(200, 200, 230)",
                         gridcolor="white",
                         showbackground=True,
                         zerolinecolor="white",),
                    yaxis = dict(
                        backgroundcolor="rgb(230, 200,230)",
                        gridcolor="white",
                        showbackground=True,
                        zerolinecolor="white"),
                    zaxis = dict(
                        backgroundcolor="rgb(230, 230,200)",
                        gridcolor="white",
                        showbackground=True,
                        zerolinecolor="white",),),
                    width=700,
                    margin=dict(
                    r=10, l=10,
                    b=10, t=10)
                  )
fig.update_layout(
    scene = dict(
        xaxis = dict(nticks=8, range=[x_inicio-(x_final*0.05),x_final],),
                     yaxis = dict(nticks=8, range=[y_inicio - (y_final*0.05),y_final],),
                     zaxis = dict(nticks=10, range=[0,h+j],),),
    width=1000,
    margin=dict(r=20, l=10, b=10, t=10))
fig.update_layout(
    scene = dict(
        aspectratio = dict(
            x = x_final/50,
            y = y_final/50,
            z = (h+j)/50
        )
    ),
    margin = dict(
        t = 20,
        b = 20,
        l = 20,
        r = 20
    )
)
fig.update_layout(scene = dict(
                    xaxis_title='X',
                    yaxis_title='Y',
                    zaxis_title='Z'),
                    width=900,
                    margin=dict(r=20, b=10, l=10, t=10))
fig.show()

data = {'Variable':["N° pozos", "Diámetro pozo", "Altura banco", 'Largo malla', 'Ancho malla', "Metros perforados", "Densidad Roca", 
                    "Densidad Explosivo","VOD", "RQD", "Resistencia a la compresión UCS",'Resistencia a la tracción estática', 
                    'Índice de volabilidad ', "Potencia  de detonación", "Burden", "Espaciamiento", "Taco", "Pasadura", "Carga explosiva"], 
        'Valor':[n_pozos, d_pozo, h, x_size, y_size, m_per, dr, de, vod, rqd,ucs,ts, kv, pod_psi, burden, espaciamiento, Tcritico, j,c_e], 
        "Unidad": ["", "m","m","m", "m", "m", "t/m3","t/m3","m/s","%","psi","psi", "", "psi", "m", "m","m","m","m"]} 
  
df = pd.DataFrame(data) 
df 
