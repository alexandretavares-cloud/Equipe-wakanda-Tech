#include <SPI.h>
#include <MFRC522.h>
#include "DHT.h"
 
// --- Sensor DHT11 ---
#define DHTPIN 2
#define DHTTYPE DHT11
DHT dht(DHTPIN, DHTTYPE);
 
// --- Sensor KY-038 (Som) ---
// Conectado a um pino ANALÓGICO (A0) para obter a intensidade
#define SOM_PIN A0    
 
// --- Sensor LDR (Luz) ---
// Conectado a um pino ANALÓGICO (A1) para obter a intensidade
#define LDR_PIN A1    
 
// --- RFID RC522 ---
#define SS_PIN 10
#define RST_PIN 9
MFRC522 mfrc522(SS_PIN, RST_PIN);
 
// --- LEDs ---
int ledAzul = 4;     // Baixa Temperatura (Frio)
int ledVermelho = 6;  // Alta Temperatura (Quente)
int ledAmarelo = 7;   // Baixa Umidade (Seco)
int ledVerde = 8;     // Alta Umidade (Úmido)
 
// --- Buzzers ---
int buzzerTemperatura = 3; // Alerta de Temperatura
int buzzerUmidade = 13; // Alerta de Umidade
 
// --- Controle ---
bool sistemaAtivo = true;
 
// --- TAGs permitidas ---
byte tagCartao[4] = {0xC6, 0x94, 0x45, 0xAE};
byte tagChaveiro[4] = {0x63, 0x45, 0x0E, 0xF7};
 
// --- Controle de leitura ---
unsigned long ultimaLeitura = 0;
// Intervalo de 2s para leituras confiáveis do DHT11
const unsigned long intervaloLeitura = 2000;
 
// --- PARÂMETROS DE ALERTA ---
const float TEMP_FRIO = 18.0;
const float TEMP_QUENTE = 22.0;
const float UMI_SECO = 45.0;
const float UMI_UMIDO = 55.0;
 
// --- Função comparar UID ---
bool compararUID(byte *idLido, byte *idPermitido) {
    for (byte i = 0; i < 4; i++) {
        if (idLido[i] != idPermitido[i]) return false;
    }
    return true;
}
 
// --- Desligar tudo (LEDs e Buzzers) ---
void desligarTudo() {
    digitalWrite(ledAzul, LOW);
    digitalWrite(ledVermelho, LOW);
    digitalWrite(ledAmarelo, LOW);
    digitalWrite(ledVerde, LOW);
    noTone(buzzerTemperatura);
    noTone(buzzerUmidade);
}
 
void setup() {
    Serial.begin(9600);
    dht.begin();
    SPI.begin();
    mfrc522.PCD_Init();
 
    // Configuração dos pinos digitais de saída
    pinMode(ledAzul, OUTPUT);
    pinMode(ledVermelho, OUTPUT);
    pinMode(ledAmarelo, OUTPUT);
    pinMode(ledVerde, OUTPUT);
    pinMode(buzzerTemperatura, OUTPUT);
    pinMode(buzzerUmidade, OUTPUT);
 
    Serial.println("Sistema de Monitoramento e Controle Ativado.");
    Serial.println("Aproxime a TAG RFID para Ativar/Desativar.");
}
 
void loop() {
    unsigned long agora = millis();
 
    // --------------------------------
    // I. LÓGICA RFID (Controle de Ativação)
    // --------------------------------
    if (mfrc522.PICC_IsNewCardPresent() && mfrc522.PICC_ReadCardSerial()) {
        byte *id = mfrc522.uid.uidByte;
 
        if (compararUID(id, tagCartao) || compararUID(id, tagChaveiro)) {
            sistemaAtivo = !sistemaAtivo; // Inverte o estado
 
            if (sistemaAtivo) {
                Serial.println("✅ Sistema ativado!");
                tone(buzzerTemperatura, 3200, 200); 
            } else {
                Serial.println("⛔ Sistema desativado!");
                desligarTudo();
                tone(buzzerUmidade, 3200, 200); 
            }
            delay(2000); 
        } else {
            Serial.println("❌ TAG nao reconhecida!");
            tone(buzzerTemperatura, 4000, 500); 
        }
        mfrc522.PICC_HaltA();
    }
 
    // Sai do loop se o sistema estiver desativado
    if (!sistemaAtivo) return;
 
    // --------------------------------
    // II. LEITURA E CONTROLE DOS SENSORES
    // --------------------------------
 
    if (agora - ultimaLeitura >= intervaloLeitura) {
        ultimaLeitura = agora;
 
        // Leitura dos sensores
        float temperatura = dht.readTemperature();
        float umidade = dht.readHumidity();

        // Leitura SOM (KY-038): Retorna valor de 0 a 1023
        int intensidade_sonora_raw = analogRead(SOM_PIN); 

        // ✅ CORREÇÃO AQUI: Mapeia o valor lido (0-1023) para o intervalo (0-52)
        int intensidade_sonora = map(intensidade_sonora_raw, 0, 1023, 0, 52);
        
        // Leitura LUZ (LDR): Retorna valor de 0 a 1023
        int intensidade_luz = analogRead(LDR_PIN);    
 
        if (isnan(temperatura) || isnan(umidade)) {
            Serial.println("❌ Falha na leitura do DHT11! Tentando novamente...");
            return; 
        }
 
        // CONFIGURAÇÃO PARA ENVIO DE DADOS AO PYTHON
        // NOVO FORMATO: Temperatura,Umidade,Som,Luz
        Serial.println(String(temperatura) + "," + String(umidade) + "," + String(intensidade_sonora) + "," + String(intensidade_luz));
 
        // Impressão para debug no Serial Monitor
        Serial.print("🌡️ Temperatura: "); Serial.print(temperatura); Serial.println(" C");
        Serial.print("💧 Umidade: "); Serial.print(umidade); Serial.println(" %");
        Serial.print("🔊 Som: "); Serial.println(intensidade_sonora);
        Serial.print("💡 Luz: "); Serial.println(intensidade_luz);
        Serial.println("-----------------------");
 
        // --- 1. BUZZER E LED TEMPERATURA ---
        if (temperatura < TEMP_FRIO) {    
            digitalWrite(ledAzul, HIGH); digitalWrite(ledVermelho, LOW); tone(buzzerTemperatura, 2400);   
        }
        else if (temperatura > TEMP_QUENTE) {    
            digitalWrite(ledAzul, LOW); digitalWrite(ledVermelho, HIGH); tone(buzzerTemperatura, 2400);   
        }
        else {    
            digitalWrite(ledAzul, LOW); digitalWrite(ledVermelho, LOW); noTone(buzzerTemperatura);
        }
 
        // --- 2. BUZZER E LED UMIDADE ---
        if (umidade < UMI_SECO) {    
            digitalWrite(ledAmarelo, HIGH); digitalWrite(ledVerde, LOW); tone(buzzerTemperatura, 2400);
        }
        else if (umidade > UMI_UMIDO) {    
            digitalWrite(ledAmarelo, LOW); digitalWrite(ledVerde, HIGH); tone(buzzerTemperatura, 2400);
        }
        else {    
            digitalWrite(ledAmarelo, LOW); digitalWrite(ledVerde, LOW); noTone(buzzerUmidade);
        }
    }
}
