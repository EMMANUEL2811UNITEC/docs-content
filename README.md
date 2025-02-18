# docs-content

## What Is this Repository?

This repository hosts the content for Arduino's documentation website. The content needs to be written in Markdown and will be converted to HTML automatically during the build process.

The production website is available at: <https://docs.arduino.cc>

## How Can I Contribute?

Contributing by creating content or suggesting changes to existing content can be done by making **pull requests**.

You start by forking the repository or by creating a new branch if you have write access to this repo. Create a new branch based on main and name it according to what you will create prefixed with your GitHub username and a slash (e.g. `sebromero/wifi-tutorial`). Read in the section below how to add different types of new content.

When you're done with a draft you can create a pull request. This will give the content team the possibility to review it and leave comments or request changes. During this review process, you can continue to push commits to the same branch. They will show up in the pull request automatically.

Once the pull request gets approved and merged into main, the content will be deployed to the live server.

## Fixing Bugs and Typos

If you find a mistake in the content, you need to locate the corresponding file to fix it and create a pull request. Here is how to locate the content.

### Products

- If you have found an issue in a hardware product-specific tutorial they are located according to the following pattern:
  `/content/hardware/[product-family]/[product-type]/[product]/tutorials/[tutorial-name]/[content-file].md`

- If you have found an issue in a hardware product's datasheet they are located according to the following pattern:
  `/content/hardware/[product-family]/[product-type]/[product]/datasheet/datasheet.md`

- If you have found an issue in a hardware product's description they are located according to the following pattern:
  `/content/hardware/[product-family]/[product-type]/[product]/product.md`

- If you have found an issue in a hardware product's tech specs table they are located according to the following pattern:
  `/content/hardware/[product-family]/[product-type]/[product]/tech-specs.yml`

- If you have found an issue in a hardware product's features they are located according to the following pattern:
  `/content/hardware/[product-family]/[product-type]/[product]/features.md`

### Software

- If you have found an issue in a software product's tutorial they are located according to the following pattern:
  `/content/software/[product-name]/tutorials/(tutorial-subfolder)/[tutorial-name]/[content-file].md`

## Adding Content

### Referencing Content From Other Folders

The build system supports symlinks. This allows the inclusion of content in multiple places. For example, if there is a tutorial that works for different boards, it can be written once and included in different places. On Unix the `ln` command can be used for that.

For example, if we want a tutorial that lives here `content/tutorials/generic/basic-servo-control` to show up on the Nano 33 BLE product page, we can link it as follows. First, open a shell and navigate to the tutorials folder of the product. e.g. `cd content/hardware/03.nano/boards/nano-33-ble/tutorials/`. Then create a symlink with a relative path to the tutorial. e.g. `ln -s ../../../../../tutorials/generic/basic-servo-control basic-servo-control`. This will create a symbolic link to that directory without duplicating it. Any change can be made in either location. They will be applied to the source file in both cases.

#### Adding Symlinks on Windows

To create symlinks using Windows OS, follow the below steps:

- Start a terminal (CMD) as admin.
- Navigate to the folder you want the symlinks. For example, to create a link for the UNO board, navigate to `docs.arduino.cc\content\hardware\02.hero\boards\uno-rev3\tutorials`.
- To create a symlink, you will need to run a command akin to:

```cmd
mklink AnalogInput "..\..\..\..\..\built-in-examples\03.analog\AnalogInput"
```

> The `..\..\` needs to match the location of the original file. Each `..\` is a step up the directory.

Note that when creating the symlink, you will not see a file in VS code, but when you commit changes it will be recognized.

On success, the following is printed:

```cmd
symbolic link created for AnalogInput <<===>> ..\..\..\..\..\built-in-examples\03.analog\AnalogInput
```

### Including Code Snippets

Code snippets can be included by using the triple backticks syntax e.g. ` ```arduino` followed by the code and three closing backticks. The following syntaxes are supported:

```
arduino, bash, markup, clike, c, cpp, css, css-extras, javascript, jsx, js-extras, coffeescript, diff, git, go, graphql, handlebars, json, less, makefile, markdown, objectivec, ocaml, python, reason, sass, scss, sql, stylus, tsx, typescript, wasm, yaml
```

### Including Code Blocks fetching GitHub pages

CodeBlocks are custom components that can be added directly in the Markdown on docs-content.

Using this component, the code block will be fetched directly from GitHub pages.

Syntax:

` <CodeBlock url=”https://github.com/example” className="{language}"/>`

Broken URLs will show an error alert. URL must be in the GitHub domain and must be public.

## Previewing Changes

Whenever you create a Pull Request (PR) and the label `preview` is assigned to it, a preview is created and updated for every commit, as explained at: https://github.com/arduino/docs-content/pull/1931

## License

![](https://i.creativecommons.org/l/by-sa/3.0/88x31.png)

Please note that your contribution to the Arduino Documentation is licensed under a Creative Commons Attribution-Share Alike 4.0 License. see https://creativecommons.org/licenses/by-sa/4.0/



// Tested on ESP8266 and NodeMCU



#include <ESP8266WiFi.h>

#define PORT 80

const char WiFiAPPSK[] = "ArduinoPass";     //Arduino password

//Setup wifi data
const char* ssid = "Your WiFi ssid";
const char* password = "Your WiFi password";

IPAddress ip(192,168,0,155);              //Set IP
IPAddress gateway(192,168,0,1);           //Set Gateway
IPAddress subnet(255,255,255,0);          //Set Subnet

WiFiServer server(PORT);                  // create a server at port 80
String request;                           // stores the HTTP request

int Status = WL_IDLE_STATUS;
int tries = 0;
int pin0 = D0; // D0 is for NodeMCU, please change it to match the Arduino you use


void setup()
{

  Serial.begin(115200);
  Serial.println("Starting arduino...");
  delay(50);

  pinMode(pin0, INPUT);

  Serial.println("Status : " + String(Status));

  WiFi.mode(WIFI_AP_STA);
  WiFi.begin(ssid, password);
  WiFi.config(ip, gateway, subnet);

  Serial.print("Connecting");

  while(WiFi.status() != WL_CONNECTED){
    Serial.print(".");
    delay(500);
    tries = tries + 1;
    if(tries > 20)
      break;
  }

  if(WiFi.status() == WL_CONNECT_FAILED | WiFi.status() == WL_CONNECTION_LOST){
    Serial.println("Failed to connect.");
  }else{
    Serial.println("Connected");
    Serial.println("IP Address : ");
    Serial.println(WiFi.localIP());
  }

  delay(100);
  Serial.println("Starting WiFi server.");
  server.begin();
  Serial.println("Server setup finished");

  delay(200);

  Serial.println("All done");
}

void loop()
{
  HtmlPage();
  delay(10);
}

void HtmlPage(){
  WiFiClient client = server.available();  // try to get client
  if (client) {  // got client?
    boolean currentLineIsBlank = true;

    while (client.connected()) {
      if (client.available()) {   // client data available to read
        char c = client.read(); // read 1 byte (character) from client
        request += c;  // save the HTTP request 1 char at a time
        // last line of client request is blank and ends with \n
        // respond to client only after last line received
        if (c == '\n' && currentLineIsBlank) {
          // Send a standard http response header
          client.println("HTTP/1.1 200 OK");
          client.println("Content-Type: text/html");
          client.println("Connection: close");
          client.println();
          // Send web page
          client.println("<!DOCTYPE html>");
          client.println("<html>");
          client.println("<head>");
          client.println("</head>");
          client.println("<body>");
          client.println("<form method=\"get\">");
          ReadData(client);
          client.println("</form>");
          client.println("</body>");
          client.println("</html>");
          request = "";    // finished with request, empty string
          break;
          }
        // every line of text received from the client ends with \r\n
        if (c == '\n') {
          // last character on line of received text
          // starting new line with next character read
          currentLineIsBlank = true;
        }
        else if (c != '\r') {
          // a text character was received from client
          currentLineIsBlank = false;
        }
      }
    }
    delay(10);      // give the web browser time to receive the data
    client.stop(); // close the connection
  } 
}

void ReadData(WiFiClient cl){
  //You can also use analogRead function
  //<br/> is used to create new line
  //id is used by application to read data
  cl.println("<br/><a id=\"pin0\">" + String(digitalRead(pin0)) + "</a>")
}

void setupWiFi()
{
  uint8_t mac[WL_MAC_ADDR_LENGTH];
  WiFi.softAPmacAddress(mac);
  String macID = String(mac[WL_MAC_ADDR_LENGTH - 2], HEX);
                 String(mac[WL_MAC_ADDR_LENGTH - 1], HEX);
  macID.toUpperCase();
  String AP_NameString = "LED_Strip_WIFI";

  char AP_NameChar[AP_NameString.length() + 1];
  memset(AP_NameChar, 0, AP_NameString.length() + 1);

  for (int i=0; i<AP_NameString.length(); i++)
    AP_NameChar[i] = AP_NameString.charAt(i);

  WiFi.softAP(AP_NameChar, WiFiAPPSK);
}
