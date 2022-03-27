# AppProcess

### Introduction

app_process is an executable program native to Android, located in the /system/bin directory, and the zygote process is started by this executable file.

AppProcess is a library which run app_process, transfer data between Android app & app_process to give the Android app a root privileges.


### Installation

https://jitpack.io/#ThinhVu/appProcess

### Prerequisites:
```
implementation 'com.google.code.gson:gson:2.8.5'
```

### Example
```java
import com.google.gson.JsonObject;

// This block of code will be run in your main process
public class Example {
    public static void Run() throws Exception {
        print();
        echo();
    }

    // this example show you how to send message from parent process to app_process
    // without caring about response data
    public static void print() throws Exception {
        String publicSourceDir = ""; // (applicationContext instance).getApplicationInfo().publicSourceDir;
        AppProcessHost apHost = AppProcessHost.create(PrintProcess.class, publicSourceDir);
        
        apHost.send("Hello there");

        // above line of code is a short hand for
        JsonObject jo = new JsonObject();
        jo.addProperty("message", "Hello there");
        apHost.send(jo);
    }

    // this example show you how to send json object from parent process to app_process
    // and received response data.
    public static void echo() throws Exception {
        String publicSourceDir = ""; // (applicationContext instance).getApplicationInfo().publicSourceDir;
        AppProcessHost apHost = AppProcessHost.create(EchoProcess.class, publicSourceDir);
        JsonObject jo = new JsonObject();
        jo.addProperty("name", "Joey");
        apHost.registerLog(/*String*/ s -> { /* do something with log contenr */ });
        apHost.send(jo, (res) -> {
            if (res.has("echo")) {
                String echoMessage = res.get("echo").getAsString();
                // do stuff with echo Message
            }
        });
    }
}

// This block of code will be run on EchoProcess process
class PrintProcess {
    public static void main(String[] args) {
        AppProcess.start((payload, res) -> {
            if (payload.has("message")) {
                // plain string in app_process_host will be store in "message" prop of payload
                String message = payload.get("message").getAsString();
                // do something with message
             
                // in Example.print apHost.send get called without callback
                // calling res.json(data) in this method doesn't have any meaning   
            }
        });
    }
}

// This block of code will be run on ValidateProcess process
class EchoProcess {
    public static void main(String[] args) {
        AppProcess.start((payload, res) -> {
            if (payload.has("name")) {
                String name = payload.get("name").getAsString();
                JsonObject jo = new JsonObject();
                jo.addProperty("echo", "Echo: " + name);
                
                // calling res.json return json object to parent process
                res.json(jo);   
            }
        });
    }
}

```
