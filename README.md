# Lecture-5---Lab-1-FreeRTOS-Queue
<img width="1613" height="819" alt="capture5" src="https://github.com/user-attachments/assets/98dfd676-75fe-4734-8591-c25ff2a28c16" />

ESP32 FreeRTOS Queue Assignment
Purpose
In this assignment, I worked with FreeRTOS on an ESP32 to understand how tasks can communicate with each other using a queue.

How It Works
I created two tasks: a Sensor Task and a Display Task.

The Sensor Task starts with a temperature value of 20.
Every second, it increases the value by 1 and sends it to the queue.
The queue can hold up to 5 integer values.
The Display Task receives the values from the queue and prints them in the Serial Monitor.
The temperature is simulated in the code, so it does not come from a real sensor.

Serial Monitor
When I uploaded the code to the ESP32, I could see the temperature values appearing in the Serial Monitor. The values started at 21°C and increased by 1 every second.

Reflection
This assignment helped me understand how two FreeRTOS tasks can work separately and still share information. I learned that a queue is useful because it provides a way for one task to send data and another task to receive it.

#include <Arduino.h>

// Declare the queue
QueueHandle_t temperatureQueue;


// --------------------------------------------------
// Sensor Task
// --------------------------------------------------

void sensorTask(void *parameter)
{
  int temperature = 20;

  while (1)
  {
    // Send temperature to queue
    xQueueSend(temperatureQueue, &temperature, portMAX_DELAY);

    // Increase temperature
    temperature++;

    vTaskDelay(pdMS_TO_TICKS(1000));
  }
}


// --------------------------------------------------
// Display Task
// --------------------------------------------------

void displayTask(void *parameter)
{
  int receivedTemperature;

  while (1)
  {
    // Receive temperature from queue
    if (xQueueReceive(temperatureQueue, &receivedTemperature, portMAX_DELAY) == pdPASS)
    {
      Serial.print("Temperature: ");
      Serial.print(receivedTemperature);
      Serial.println(" C");
    }
  }
}


// --------------------------------------------------
// Setup
// --------------------------------------------------

void setup()
{
  Serial.begin(115200);

  // Create queue for 5 integers
  temperatureQueue = xQueueCreate(5, sizeof(int));

  if (temperatureQueue == NULL)
  {
    Serial.println("Queue creation failed!");
    return;
  }

  // Create Sensor Task
  xTaskCreate(
    sensorTask,
    "Sensor Task",
    2048,
    NULL,
    1,
    NULL
  );

  // Create Display Task
  xTaskCreate(
    displayTask,
    "Display Task",
    2048,
    NULL,
    1,
    NULL
  );
}


// --------------------------------------------------
// Loop
// --------------------------------------------------
