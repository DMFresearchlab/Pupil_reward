import time
from time import perf_counter

import pyglet
from pyglet.gl import gl  #funciones OpenGL
from pyglet import clock
from pyglet.window import key #para teclado
from psychopy import gui
import numpy, random

import csv
import zmq
from zmq.utils.monitor import recv_monitor_message # from zmq_tools
import msgpack as serializer
import threading


nensayos=360   #total de imagenes (la 1º mitad animales la 2º mitad objetos)
ctrial = 0
time1 = 0.0
time2 = 0.0
esperatecla=0
inicio=0
t_inic=0
t_inicf=0.0
sujeto="John Doe"
rdt1=0;     ##Tiempo inter-estímulo mínimo (ms)
rdt2=0;     ##Tiempo inter-estímulo máximo (ms)
t_fix=0.0;  #Tiempo de fijacion
t_pres=0.0; #Tiempo presentacion estímulo
Nimage='kk'
ima=-1
cat=-1
resp=-1
ett=0
pupilTime=0.0
fin=0

##genera serie experimental

SECEX=[]
aux1=[]

for x in range(nensayos):
    SECEX.append(0)
    aux1.append(0)

for x in range(nensayos):
    cont=0
    while (cont < 1 ):
        num1= random.randint(0, nensayos-1);
        if aux1[num1] < 1:
            aux1[num1]=1
            SECEX[x]=num1
            cont=cont+1
cont=0

# cuadro de dialogo
myDlg = gui.Dlg(title="Setting experiment")
myDlg.addField('ID Sujeto:', ' ')
myDlg.addField('Tamaño estímulo (px):', '400')
myDlg.addField('Tiempo fijación (ms):', '1000')
myDlg.addField('Tiempo presentación estímulo (ms):', '3000')
myDlg.addField('Tiempo inter-estímulo mínimo (ms):', '1500')
myDlg.addField('Tiempo inter-estímulo máximo (ms):', '2000')
myDlg.show()

#carga variables del dialog box
sujeto=str(myDlg.data[0])
width=int(myDlg.data[1])
height=int(myDlg.data[1])
t_fix=float(myDlg.data[2])/1000
t_pres=float(myDlg.data[3])/1000
rdt1=int(myDlg.data[4])
rdt2=int(myDlg.data[5])

##crea fichero datos
fecha=time.strftime("%d_%m_%y_")
hora=time.strftime("%H%M") 
print(fecha)
fileName = sujeto + fecha + hora
dataFile = open(fileName+'.csv', 'w')  
dataFile.write('trial,t_pupil,t_inter,t_fix,t_pres,imagen,imgN,cat,resp\n')

# define color de fondo
background_color = (.5, .5, .5, 1)

##busca e imprime pantallas disponibles
display = pyglet.canvas.get_display()
screens = display.get_screens()
for screen in screens:
    print(screen)
    
## conecta eyetracker
ctx = zmq.Context()
requester = ctx.socket(zmq.REQ)
requester.connect('tcp://127.0.0.1:50020')
# Request 'SUB_PORT' for reading data
requester.send_string('SUB_PORT') 
sub_port = requester.recv_string()

## pupil time
requester.send_string('t')
pupilTime=requester.recv_string()
print(pupilTime)


#crea ventana pyglet
window = pyglet.window.Window(fullscreen=True, screen=screens[1])

# aplica color de fondo
gl.glClearColor(*background_color)

#carga imagenes fijas
image = pyglet.resource.image('img_ini.jpg')
image1 = pyglet.resource.image("void.jpg")
image2 = pyglet.resource.image("p_fijac.jpg")
image3 = pyglet.resource.image("img_tec.jpg")
image4 = pyglet.resource.image("img_fin.jpg")

def update(dt):
    count=0
      
@window.event
def on_draw():    
    global image
    global time1,time2
    global esperatecla
    global inicio
    global t_inic
    global t_inicf
    global requester
    global ett
    global pupilTime
    global fin

    t_inicf=float(t_inic)/1000    
    time2=perf_counter()
    esperatecla=0
    window.clear()
    
    if inicio==0:       ##presenta imagen inicio
        image.blit((window.width/2)-width/2, (window.height/2)-height/2, width
                   =width, height=height)
        esperatecla=1
        
    if fin==1:       ##presenta imagen inicio
        image4.blit((window.width/2)-width/2, (window.height/2)-height/2, width
                    =width, height=height)
        esperatecla=0

    if time2-time1<t_inicf and inicio==1 and fin==0:     ##presenta imagen vacia entre rdt1 y rdt2
        image1.blit((window.width/2)-width/2, (window.height/2)-height/2, width
                    =width, height=height)

    if (time2-time1>=t_inicf and time2-time1<t_fix+t_inicf and inicio==1 
        and ett==0):     ##captura tiempo eyetracker
        ## pupil time
        requester.send_string('t')
        pupilTime=requester.recv_string()
        print(pupilTime)
        ett=1 

    if time2-time1>=t_inicf and time2-time1<t_fix+t_inicf and inicio==1 and fin==0:     ##presenta punto fijacion 
        image2.blit((window.width/2)-width/2, (window.height/2)-height/2, width=width, height=height)    
       
    if time2-time1>=t_fix+t_inicf and time2-time1<t_pres+t_fix+t_inicf and inicio==1 and fin==0:     ##presenta imagen animal/objeto
        image.blit((window.width/2)-width/2, (window.height/2)-height/2, width=width, height=height)
        
    if time2-time1>=t_pres+t_fix+t_inicf and inicio==1 and fin==0:    ##presenta imagen respuesta
        image3.blit((window.width/2)-width/2, (window.height/2)-height/2, width=width, height=height)
        esperatecla=1

   
@window.event
def on_key_press(symbol, modifiers):
    global ctrial
    global image
    global time1
    global esperatecla
    global inicio
    global t_inic
    global dataFile
    global Nimage
    global ima
    global cat
    global resp
    global requester
    global ett
    global sujeto
    global pupilTime
    global fin
    
    if symbol == key.RIGHT and esperatecla==1 and inicio==1:
        resp=1
        dataFile.write('%i,%s,%i,%i,%i,%s,%i,%i,%i\n'%(ctrial,pupilTime,t_inic,t_fix*1000,t_pres*1000,Nimage,ima,cat,resp))
        time1=perf_counter()
        if ctrial==nensayos:
            fin=1
            return
        if SECEX[ctrial] < nensayos/2:
            Nimage="AN"+str(SECEX[ctrial]+1)+"_720.jpg"
            ima=SECEX[ctrial]+1
            cat=0
        if SECEX[ctrial]>= nensayos/2:
            Nimage="OB"+str(SECEX[ctrial]-int(nensayos/2)+1)+"_720.jpg"
            ima=SECEX[ctrial]-int(nensayos/2)+1
            cat=1
        image = pyglet.resource.image(Nimage)
        ctrial=ctrial+1
        
    elif symbol == key.LEFT and esperatecla==1 and inicio==1:
        resp=0
        dataFile.write('%i,%s,%i,%i,%i,%s,%i,%i,%i\n'%(ctrial,pupilTime,t_inic,t_fix*1000,t_pres*1000,Nimage,ima,cat,resp))
        time1=perf_counter()
        if ctrial==nensayos:
            fin=1
            return
        if SECEX[ctrial] < nensayos/2:
            Nimage="AN"+str(SECEX[ctrial]+1)+"_720.jpg"
            ima=SECEX[ctrial]+1
            cat=0
        if SECEX[ctrial]>= nensayos/2:
            Nimage="OB"+str(SECEX[ctrial]-int(nensayos/2)+1)+"_720.jpg"
            ima=SECEX[ctrial]-int(nensayos/2)+1
            cat=1
        image = pyglet.resource.image(Nimage)       
        ctrial=ctrial+1
        
    elif symbol == key.SPACE and esperatecla==1 and inicio==0: ## es la 1º imagen         
        inicio=1
        time1=perf_counter()
        if SECEX[ctrial] < nensayos/2:
            Nimage="AN"+str(SECEX[ctrial]+1)+"_720.jpg"
            ima=SECEX[ctrial]+1
            cat=0
        if SECEX[ctrial]>= nensayos/2:
            Nimage="OB"+str(SECEX[ctrial]-int(nensayos/2)+1)+"_720.jpg"
            ima=SECEX[ctrial]-int(nensayos/2)+1
            cat=1
        image = pyglet.resource.image(Nimage)
        ctrial=ctrial+1
        requester.send_string('R '+ sujeto)   ## comienza registro eyetracker
        requester.recv_string()
        
    t_inic=random.randint(rdt1, rdt2) ##calcula intervalo inter-estimulo
    ett=0
        
pyglet.clock.schedule_interval(update, 1/120)

pyglet.app.run()
dataFile.close()
requester.send_string('r '+ sujeto)   ## termina registro eyetracker
requester.recv_string()
