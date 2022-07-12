# Pre_practica_Gheorghe_Coroi

/*
    ------ Waspmote Pro Code Example --------

    Explanation: This is the basic Code for Waspmote Pro

    Copyright (C) 2016 Libelium Comunicaciones Distribuidas S.L.
    http://www.libelium.com

    This program is free software: you can redistribute it and/or modify
    it under the terms of the GNU General Public License as published by
    the Free Software Foundation, either version 3 of the License, or
    (at your option) any later version.

    This program is distributed in the hope that it will be useful,
    but WITHOUT ANY WARRANTY; without even the implied warranty of
    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
    GNU General Public License for more details.

    You should have received a copy of the GNU General Public License
    along with this program.  If not, see <http://www.gnu.org/licenses/>.
*/

// Put your libraries here (#include ...)
#include <WaspWIFI_PRO_V3.h>
#include <WaspFrame.h>
#include <WaspSensorGas_Pro.h>

bmeGasesSensor  bme;

float temperature;  // Stores the temperature in ºC
float humidity;   // Stores the realitve humidity in %RH
float pressure;   // Stores the pressure in Pa

// choose socket (SELECT USER'S SOCKET)
///////////////////////////////////////
uint8_t socket = SOCKET0;
///////////////////////////////////////

// WiFi AP settings (CHANGE TO USER'S AP)
///////////////////////////////////////
char SSID[] = "LANCOMBEIA";
char PASSW[] = "beialancom";
///////////////////////////////////////

// define variables
uint8_t error;
uint8_t status;
unsigned long previous;


// choose HTTP server settings
///////////////////////////////////////
char type[] = "http";
char host[] = "82.78.81.171";
uint16_t port = 80;
///////////////////////////////////////

// define the Waspmote ID
char moteID[] = "Gheorghe_Cor";

// de calibrat senzorii la portul respectiv



void setup()
{
  // put your setup code here, to run once:
  USB.ON();
  USB.println(F("Temperature, humidity and pressure sensor example"));

  USB.println(F("Start program"));
  USB.println(F("Start program"));

  //////////////////////////////////////////////////
  // 1. Switch ON the WiFi module
  //////////////////////////////////////////////////
  error = WIFI_PRO_V3.ON(socket);

  if (error == 0)
  {
    USB.println(F("1. WiFi switched ON"));
  }
  else
  {
    USB.println(F("1. WiFi did not initialize correctly"));
  }
  //////////////////////////////////////////////////
  // 2. Reset to default values
  //////////////////////////////////////////////////
  error = WIFI_PRO_V3.resetValues();

  if (error == 0)
  {
    USB.println(F("2. WiFi reset to default"));
  }
  else
  {
    USB.print(F("2. WiFi reset to default error: "));
    USB.println(error, DEC);
  }

  //////////////////////////////////////////////////
  // 3. Configure mode (Station or AP)
  //////////////////////////////////////////////////
  error = WIFI_PRO_V3.configureMode(WaspWIFI_v3::MODE_STATION);

  if (error == 0)
  {
    USB.println(F("3. WiFi configured OK"));
  }
  else
  {
    USB.print(F("3. WiFi configured error: "));
    USB.println(error, DEC);
  }

  // get current time
  previous = millis();


  //////////////////////////////////////////////////
  // 4. Configure SSID and password and autoconnect
  //////////////////////////////////////////////////
  error = WIFI_PRO_V3.configureStation(SSID, PASSW, WaspWIFI_v3::AUTOCONNECT_ENABLED);

  if (error == 0)
  {
    USB.println(F("4. WiFi configured SSID OK"));
  }
  else
  {
    USB.print(F("4. WiFi configured SSID error: "));
    USB.println(error, DEC);
  }


  if (error == 0)
  {
    USB.println(F("5. WiFi connected to AP OK"));

    USB.print(F("SSID: "));
    USB.println(WIFI_PRO_V3._essid);

    USB.print(F("Channel: "));
    USB.println(WIFI_PRO_V3._channel, DEC);

    USB.print(F("Signal strength: "));
    USB.print(WIFI_PRO_V3._power, DEC);
    USB.println("dB");

    USB.print(F("IP address: "));
    USB.println(WIFI_PRO_V3._ip);

    USB.print(F("GW address: "));
    USB.println(WIFI_PRO_V3._gw);

    USB.print(F("Netmask address: "));
    USB.println(WIFI_PRO_V3._netmask);

    WIFI_PRO_V3.getMAC();

    USB.print(F("MAC address: "));
    USB.println(WIFI_PRO_V3._mac);
  }
  else
  {
    USB.print(F("5. WiFi connect error: "));
    USB.println(error, DEC);

    USB.print(F("Disconnect status: "));
    USB.println(WIFI_PRO_V3._status, DEC);

    USB.print(F("Disconnect reason: "));
    USB.println(WIFI_PRO_V3._reason, DEC);

    /*
      enum ConnectionFailureReason{
        REASON_INTERNAL_FAILURE = 1,
        REASON_AUTH_NO_LONGER_VALID = 2,
        REASON_DEAUTH_STATION_LEAVING = 3,
        REASON_DISASSOCIATED_INACTIVITY = 4,
        REASON_DISASSOCIATED_AP_HANDLE_ERROR = 5,
        REASON_PACKET_RECEIVED_FROM_NONAUTH_STATION = 6,
        REASON_PACKET_RECEIVED_FROM_NONASSOC_STATION = 7,
        REASON_DISASSOCIATED_STATION_LEAVING = 8,
        REASON_STATION_REQUEST_REASSOC = 9,
        REASON_DISASSOCIATED_PWR_CAPABILITY_UNACCEPTABLE = 10,
        REASON_DISASSOCIATED_SUPPORTED_CHANNEL_UNACCEPTABLE = 11,
        REASON_INVALID_ELEMENT = 13,
        REASON_MIC_FAILURE = 14,
        REASON_FOUR_WAY_HANDSHAKE_TIMEOUT = 15,
        REASON_GROUP_KEY_HANDSHAKE_TIMEOUT = 16,
        REASON_ELEMENT_IN_FOUR_WAY_HANDSHAKE_DIFFERENT = 17,
        REASON_INVALID_GROUP_CIPHER = 18,
        REASON_INVALID_PAIRWISE_CIPHER = 19,
        REASON_AKMP = 20,
        REASON_UNSUPPORTED_RSNE_CAPABILITIES = 21,
        REASON_INVALID_RSNE_CAPABILITIES = 22,
        REASON_IEEE_AUTH_FAILED = 23,
        REASON_CIPHER_SUITE_REJECTED = 24,
        REASON_STATION_LOST_BEACONS_CONTINUOUSLY = 200,
        REASON_STATION_FAILED_TO_SCAN_TARGET_AP = 201,
        REASON_STATION_AUTH_FAILED_NOT_TIMEOUT = 202,
        REASON_STATION_AUTH_FAILED_NOT_TIMEOUT_NOT_MANY_STATIONS = 203,
        REASON_HANDHASKE_FAILED = 204,
      };
    */
  }
  frame.setID(moteID);
}



void loop()
{
  if (WIFI_PRO_V3.isConnected() == true)
  {
    bme.ON();
    USB.print(F("WiFi is connected OK"));
    USB.print(F(" Time(ms):"));
    USB.println(millis() - previous);

    USB.println(F("\n*** Program stops ***"));
    ///////////////////////////////
    // 3.1. Create a new Frame
    ///////////////////////////////

    // create new frame (only ASCII)
    frame.createFrame(ASCII);

    // add sensor fields
    // frame.addSensor(SENSOR_STR, "this_is_a_string");
    frame.addSensor(SENSOR_BAT, PWR.getBatteryLevel() );        //adaugare de inca 3 senzori
    frame.addSensor(SENSOR_GASES_PRO_TC, bme.getTemperature() );
    frame.addSensor(SENSOR_GASES_PRO_HUM, bme.getHumidity()   );
    frame.addSensor(SENSOR_GASES_PRO_PRES , bme.getPressure() );

    // print frame
    frame.showFrame();


    ///////////////////////////////
    // 3.2. Send Frame to Meshlium
    ///////////////////////////////

    // http frame
    error = WIFI_PRO_V3.sendFrameToMeshlium( type, host, port, frame.buffer, frame.length);

    // check response
    if (error == 0)
    {
      USB.println(F("Send frame to meshlium done"));
    }
    else
    {
      USB.println(F("Error sending frame"));
      if (WIFI_PRO_V3._httpResponseStatus)
      {
        USB.print(F("HTTP response status: "));
        USB.println(WIFI_PRO_V3._httpResponseStatus);
      }
    }
  }


  else
  {
    USB.print(F("WiFi is connected ERROR"));
    USB.print(F(" Time(ms):"));
    USB.println(millis() - previous);
  }

  delay(10000);

  ///////////////////////////////////////////
  // 1. Turn on the sensor
  ///////////////////////////////////////////

  //bme.ON();


  ///////////////////////////////////////////
  // 2. Read sensors
  ///////////////////////////////////////////

  // Read enviromental variables
  temperature = bme.getTemperature();
  humidity = bme.getHumidity();
  pressure = bme.getPressure();
  // temperature=22;
  // humidity=55;
  //pressure=77;

  // And print the values via USB
  USB.println(F("***************************************"));
  USB.print(F("Temperature: "));
  USB.print(temperature);
  USB.println(F(" Celsius degrees"));
  USB.print(F("RH: "));
  USB.print(humidity);
  USB.println(F(" %"));
  USB.print(F("Pressure: "));
  USB.print(pressure);
  USB.println(F(" Pa"));

  ///////////////////////////////////////////
  // 3. Turn off the sensor
  ///////////////////////////////////////////

  bme.OFF();


  ///////////////////////////////////////////
  // 3. Sleep
  ///////////////////////////////////////////

  // Go to deepsleep
  // After 3 minutes, Waspmote wakes up thanks to the RTC Alarm
  USB.println(F("Go to deep sleep mode..."));
  USB.ON();
  USB.println(F("Wake up!!\r\n"));

}
