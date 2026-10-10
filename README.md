# Lentes
Lentes detectores de objetos
// ======================================================
// GAFAS CON DETECCION DE OBSTACULOS
// ESP32 + HC-SR04 + LEDs + BOTON + BUZZER
// ======================================================

// ---------- SENSOR HC-SR04 ----------
const int trigPin = 18;
const int echoPin = 19;

// ---------- LEDs ----------
const int ledRojo = 25;
const int ledVerde = 26;

// ---------- BOTON ----------
const int boton = 27;

// ---------- BUZZER ----------
const int buzzer = 23;

// Variables del sensor
long duracion;
float distancia;

// Estado del sistema
bool sistemaActivo = true;

// Variables del botón
bool estadoAnteriorBoton = HIGH;
unsigned long ultimoCambioBoton = 0;

// Variables para controlar los pitidos
unsigned long ultimoPitido = 0;
bool estadoBuzzer = false;


void setup() {

  Serial.begin(115200);

  // Sensor
  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);

  // LEDs
  pinMode(ledRojo, OUTPUT);
  pinMode(ledVerde, OUTPUT);

  // Botón
  pinMode(boton, INPUT_PULLUP);

  // Buzzer
  pinMode(buzzer, OUTPUT);

  digitalWrite(trigPin, LOW);
  digitalWrite(buzzer, LOW);

  Serial.println("Sistema iniciado");
}


// ======================================================
// PROGRAMA PRINCIPAL
// ======================================================

void loop() {

  // ----------------------------------------------------
  // BOTON DE ENCENDIDO / APAGADO
  // ----------------------------------------------------

  bool estadoBoton = digitalRead(boton);

  if (estadoAnteriorBoton == HIGH &&
      estadoBoton == LOW &&
      millis() - ultimoCambioBoton > 200) {

    sistemaActivo = !sistemaActivo;

    ultimoCambioBoton = millis();

    if (sistemaActivo) {
      Serial.println("SISTEMA ENCENDIDO");
    }
    else {
      Serial.println("SISTEMA APAGADO");
    }
  }

  estadoAnteriorBoton = estadoBoton;


  // ----------------------------------------------------
  // SI EL SISTEMA ESTA APAGADO
  // ----------------------------------------------------

  if (!sistemaActivo) {

    digitalWrite(ledRojo, LOW);
    digitalWrite(ledVerde, LOW);
    digitalWrite(buzzer, LOW);

    delay(20);
    return;
  }


  // ----------------------------------------------------
  // MEDIR DISTANCIA
  // ----------------------------------------------------

  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);

  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);

  digitalWrite(trigPin, LOW);

  duracion = pulseIn(echoPin, HIGH, 30000);


  if (duracion == 0) {

    Serial.println("Sin lectura");

    digitalWrite(ledRojo, LOW);
    digitalWrite(ledVerde, LOW);
    digitalWrite(buzzer, LOW);

    delay(100);
    return;
  }


  distancia = duracion * 0.0343 / 2.0;


  Serial.print("Distancia: ");
  Serial.print(distancia);
  Serial.println(" cm");


  // ====================================================
  // CONTROL SEGUN DISTANCIA
  // ====================================================


  // ---------- MAS DE 100 cm ----------
  if (distancia > 100) {

    digitalWrite(ledVerde, HIGH);
    digitalWrite(ledRojo, LOW);

    digitalWrite(buzzer, LOW);
  }


  // ---------- ENTRE 50 Y 100 cm ----------
  else if (distancia > 50) {

    digitalWrite(ledVerde, HIGH);
    digitalWrite(ledRojo, LOW);

    // Pitido lento
    controlarPitido(700);
  }


  // ---------- ENTRE 20 Y 50 cm ----------
  else if (distancia > 20) {

    digitalWrite(ledVerde, LOW);
    digitalWrite(ledRojo, HIGH);

    // Pitido rápido
    controlarPitido(250);
  }


  // ---------- MENOS DE 20 cm ----------
  else {

    digitalWrite(ledVerde, LOW);
    digitalWrite(ledRojo, HIGH);

    // Sonido continuo
    digitalWrite(buzzer, HIGH);
  }


  delay(50);
}


// ======================================================
// FUNCION PARA GENERAR PITIDOS
// ======================================================

void controlarPitido(unsigned long intervalo) {

  if (millis() - ultimoPitido >= intervalo) {

    ultimoPitido = millis();

    estadoBuzzer = !estadoBuzzer;

    digitalWrite(buzzer, estadoBuzzer);
  }
}
