# Vumetro
creacion de vumetro proyecto personal FABLAB

  Primero que todo para la creacion de este vumetro necesitaras si o si:
  computadora (que tenga microfono)
  arduino 
  leds ws2812b
  cables
  filamento
  impresora 3D
  Metacrilato (de preferencia 3mm)
  
  Luego de tener todos los materiales para llevar esto a cabo 

  comenzamos con el diseño de la torre teniendo en cuenta que al tener los leds adentro tienen que tener alguna zona para que pase
  el cable al arduino y que quede espacioo para el metacrilato (acrilico) 

  poner cables correspondientes en el arduino (5V,Din,GND) y conecta el arduino a uno de los puertos de tu computadora.

  configurar el puerto seleccionado en el codigo de pyth0on que te dejo a continuacion
  a la vez carga el codigo en arduino al arduino 

  una vez cargado y conectado todo el arduino recibira las señales obtenidas por tu microfono de la computadora lo cual el arduino 
  tomara ciertas señales y las interpretara en distintos niveles de iluminacion en nuestro vumetro 

  hecho esto estaria listo
  
codigo python:

import serial
import pyaudio
import struct

# Configuración de la conexión serial (ajusta el puerto COM según tu configuración)
ser = serial.Serial('COM4', 9600)  # Reemplaza 'COMX' con el puerto COM correcto

# Configuración de PyAudio para capturar audio desde el micrófono
audio = pyaudio.PyAudio()
stream = audio.open(format=pyaudio.paInt16, channels=1, rate=44100, input=True, frames_per_buffer=1024)

try:
    while True:
        audio_data = stream.read(1024)  # Lee un bloque de audio
        # Calcula el nivel de audio (puedes ajustar esta lógica según tus necesidades)
        audio_level = max(struct.unpack('h' * 1024, audio_data))
        ser.write(f"{int(audio_level)}\n".encode())  # Envía el nivel de audio al Arduino
except KeyboardInterrupt:
    print("Detenido por el usuario")

# Cierra la conexión serial y el stream de audio cuando termines
ser.close()
stream.stop_stream()
stream.close()
audio.terminate()




Codigo arduino:

#include <Adafruit_NeoPixel.h>

#define NUM_LEDS 10 // Número de LEDs en la tira
#define LED_PIN 6    // Pin al que está conectada la tira de LEDs
Adafruit_NeoPixel strip = Adafruit_NeoPixel(NUM_LEDS, LED_PIN, NEO_GRB + NEO_KHZ800);

void setup() {
  strip.begin();  // Inicializar la tira de LEDs
  strip.show();   // Apagar todos los LEDs al inicio
  Serial.begin(9600); // Inicializar la comunicación serial a 9600 baudios
}

void loop() {
  if (Serial.available() > 0) {
    int audioLevel = Serial.parseInt(); // Leer el nivel de audio desde la conexión serial

    // Mapear el nivel de audio al número de LEDs encendidos
    int ledLevel = map(audioLevel, 0, 1023, 0, NUM_LEDS);

    // Mostrar el nivel de audio en la tira de LEDs
    for (int i = 0; i < NUM_LEDS; i++) {
      if (i < ledLevel) {
        strip.setPixelColor(i, strip.Color(255, 0, 0)); // Establecer el color del LED (rojo en este caso)
      } else {
        strip.setPixelColor(i, strip.Color(0, 0, 0)); // Apagar el LED
      }
    }
    strip.show(); // Actualizar la tira de LEDs
  }
}
