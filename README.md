# task2

Programming a web page to control the arms using web series api / writing arudino code related to the controller


`` <!DOCTYPE html>

<html lang="en" dir="ltr">

<head>

 <meta charset="utf-8">

 <title>speech to text in javascript</title>


 <style>
    *{
    -webkit-box-sizing: border-box;
    -moz-box-sizing: border-box;
    -ms-box-sizing: border-box;
    box-sizing: border-box;
    }
    body{
    background:linear-gradient(to right , rgb(9, 41, 167), rgb(179, 176, 176) );
    display: flex;
    align-items: center;
    justify-content: center;
    min-height: 100vh;
    color: #000;
    font-family: Arial, Helvetica, sans-serif;
    font-size: 16px;
    margin: 0;
            }
    .container {
    text-align: center;
    }
    h1 {
    font-size: 30px;
    color: rgb(191, 15, 15);
    }
    textarea {
    width: 100%;
    height: 200px;
    border-radius: 10px;
    font-size: 20px;
    margin-bottom: 10px;
    }
    button,select{
    padding: 12px 20px ;
    background: rgb(32, 189, 45);
    border: 0px;
    border-radius: 10px;
    cursor: pointer;
    color: rgb(234, 53, 170);
    }

    button:hover,select:hover {
    background: rgb(30, 215, 221);
    color: rgb(200, 210, 10);
    }
</style>

 <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@4.6.0/dist/css/bootstrap.min.css">

</head>

<body>

 <div class="container">

 <h1 class="text-center mt-5">

 Speech to Text in JavaScript

 </h1>

 <div class="form-group">

 <textarea  id="textbox" rows="6" class="form-control"></textarea>

 </div>

 <div class="form-group">

 <button id="start-btn"  class="btn btn-danger btn-block" title="" >
    Start
 </button>
<button class = "btn btn-danger btn-block" onclick="onConnectUsb()" id="connect-usb">
    connect
</button>


 
 <p id="instructions">Press the Start button to start speech or connect button to connect to serial port</p>

 </div>

 </div>

 <script src="https://ajax.googleapis.com/ajax/libs/jquery/3.4.1/jquery.min.js"></script>

 <script src="script.js"></script>
<script src="arduino.js"></script>
<script>
</script>
</body>
</html>``


``var speechRecognition = window.webkitSpeechRecognition

var recognition = new speechRecognition()

var textbox = $("#textbox")

var instructions = $("#instructions")

var content = ''

recognition.continuous = true


recognition.onstart = function() {

 instructions.text("Voice Recognition is On")

}

recognition.onspeechend = function() {

 instructions.text("No Activity")

}

recognition.onerror = function() {

 instruction.text("Try Again")

}

recognition.onresult = function(event) {

 var current = event.resultIndex;

 var transcript = event.results[current][0].transcript



 content += transcript
 onChangespeech()
 textbox.val(content)

}

$("#start-btn").click(function(event) {

 recognition.start()

})

textbox.on('input', function() {

 content = $(this).val()

})``


 `` let isConnectted = false;
      let port;
      let writer;
      var target_id;
      const enc = new TextEncoder();

      async function onChangespeech() {
        if (!isConnectted) {
          alert("you must connect to the usb in order to use this.");
          return;
        }
       
        try {
          const commandlist = content;
          const commandSplit = commandlist.split(" ")
          const command = commandSplit.slice(-1);
          const computerText = `${command}@`;
          await writer.write(enc.encode(computerText));
        } catch (e) {
          console.log(e);
        }
      }
    
      

    async function onConnectUsb() {
      try {
        const requestOptions = {
          // Filter on devices with the Arduino USB vendor ID.
          filters: [{ usbVendorId: 0x2341 }],
        };

        // Request an Arduino from the user.
        port = await navigator.serial.requestPort(requestOptions);
        await port.open({ baudRate: 115200 });
        writer = port.writable.getWriter();
        isConnectted = true;
      } catch (e) {
        console.log("err", e);
      }
    } ``

``
#include <Servo.h>
Servo gripper;
Servo wrist;
Servo elbow;
Servo shoulder;
Servo base;

double base_angle=90;
double shoulder_angle=90;
double elbow_angle=90;
double wrist_angle=90;


void setup() {
 Serial.begin(115200);
   base.attach(2);
  shoulder.attach(3); 
  elbow.attach(4);
  wrist.attach(5);
  gripper.attach(6); 

  base.write(base_angle);
  shoulder.write(shoulder_angle);
  elbow.write(elbow_angle);
  wrist.write(wrist_angle);

}



String getValue(String data, char separator, int index)
{
  int found = 0;
  int strIndex[] = {0, -1};
  int maxIndex = data.length()-1;

  for(int i=0; i<=maxIndex && found<=index; i++){
    if(data.charAt(i)==separator || i==maxIndex){
        found++;
        strIndex[0] = strIndex[1]+1;
        strIndex[1] = (i == maxIndex) ? i+1 : i;
    }
  }

  return found>index ? data.substring(strIndex[0], strIndex[1]) : "";
}

void loop() {
  
  String computerText = Serial.readStringUntil('@');
  computerText.trim();
  if (computerText.length() == 0) {
    return;
  }
  String command = getValue(computerText, ' ',0);

    if (command == "right" || command == "Right") {
      base.write(base_angle -= 20);
    }
    if (command == "left" || command == "Left" ) {
     base.write(base_angle += 20);
    }

    if (command == "top" || command == "Top") {
      shoulder.write(shoulder_angle -= 20);
    }

   if (command == "bottom"|| command == "Bottom") {
     shoulder.write(shoulder_angle += 20);
    }
    Serial.println(command);
  Serial.println("WORKING");
  delay(1000);

}
``
