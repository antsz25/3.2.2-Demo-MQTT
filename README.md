# 3.2.2-Demo-MQTT
### Autores:    
###             Vargas Vargas Jazmin Angelica 
###             Pantoja Parra Luis Enrique
###             Sanchez Santos Jesus Antonio

### Correos:   
###            l20212436@tectijuana.edu.mx
###            l20211821@tectijuana.edu.mx
###            l20211845@tectijuana.edu.mx

### Tecnológico Nacional de México (TECNM)  Intituto Tecnologico de Tijuana unidad Tomas Aquino
### Materia:  Sistemas Programables U#3
# Actividad: 3.2.2 DEMO con MQTT para conocer a detalle el STREAM Mensajería a internet
## Objetivo: Realizar una demostración con MQTT utilizando las PicoW de su equipo, asi conectado a un servidor MQTT en AWS Ubuntu.

# Codigo 

```python
#Import Modules
import time
import utime
import network  # for Wifi
from machine import Pin, I2C  # for LED and others
from umqtt.simple import MQTTClient #To manage MQTTBroker
import dht #For dht11 sensor
import ssd1306

# Global to check if time is set
global wifi_is_connected
#Connections for display
i2c = I2C(0,scl=17,sda=16)
display = ssd1306.SSD1306_I2C(128,64,i2c)
#Connection for led of PicoW
led = Pin("LED", Pin.OUT)
#Connections for DHT11
sensordht = dht.DHT11(Pin(15))
#Values for network
ssid = 'TecNM_ITT'
password = ''

def wifi_connect(ssid, password):
    global wifi_is_connected  # Declare wifi_is_connected as a global variable
    # Connect to WiFi
    wlan = network.WLAN(network.STA_IF)
    wlan.active(True)
    wlan.connect(ssid, password)

    max_wait = 15
    while max_wait > 0:
        if wlan.status() < 0 or wlan.status() >= 3:
            break
        max_wait -= 1
        time.sleep(5)
    if wlan.status() != 3: #Display error of connection
        display.fill(0)
        display.text("Network Failed",0,32)
        display.show()
        wifi_is_connected = False
        led.off()
    else: #Succesful connection
        wifi_is_connected = True
        status = wlan.ifconfig()
        led.on()
def ReturnTemp(): #Return temperature of dht11
    time.sleep(1)
    sensordht.measure()
    temp = sensordht.temperature()
    return temp
def ReturnHumidity(): #Return humidity of dht11
    time.sleep(1)
    sensordht.measure()
    hum = sensordht.humidity()
    return hum
def DisplayTempAndHumidity(): #Display values from dht11 in the oled display
    try:
        time.sleep(1)
        temp = ReturnTemp()
        hum = ReturnHumidity()
        display.fill(0)
        display.text('Temperature: ' + str(temp)+' C',0,10)
        display.text('Humidity: ' +str(hum)+'%',0,32)
        display.show()
    except OSError as e:
        print('Failed to read sensor')
def ConnectToMQTTBroker(): #Connect the raspberry with mqttbroker
    client = MQTTClient(client_id=b"RaspBerrySensor", #Name of the broker
    server=b"52.72.132.80", #Server
    port=1883, #Port
    keepalive=7200) 
    client.connect() #Connect client
    return client
def publish(topic,value): #Publish topics and values on the MQTTBroker
    print(topic)
    print(value)
    client.publish(topic,value)
    print("publish done")
while True:
    wifi_connect(ssid,password)
    if wifi_is_connected:
        client = ConnectToMQTTBroker()
        DisplayTempAndHumidity()
        publish("picow/Temperatura",str(ReturnTemp()))
        publish("picow/Humedad",str(ReturnHumidity()))
        utime.sleep(10)
    else:
        utime.sleep(2)
        display.fill(0)
        display.show()

```

![Solucion](https://github.com/JAZMIN2021/EquipoSP/assets/79472215/19399201-da47-4323-a171-0c8aa9b79823)

![](https://github.com/JAZMIN2021/EquipoSP/assets/79472215/6cc09fd5-1832-4e33-bf8a-2968ceb399e0)
