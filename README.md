# Electron work around for transparent acrylic window
![image](https://user-images.githubusercontent.com/60309361/221607891-5aa47deb-edf3-4eec-a009-d1b6f4818b3f.png)

> **Warning**: this is just a work around but not an exact acrylic window for electron js

#  Modules required
 - electron js
 - fs

# How it works
we are using the electron's main renderer to listen to changes in position, minimize, maximize and focus events then send the position and size of the window. Then the  render process's `preload.js` will listen to them via the IPC-Main and respectively change the image's position and size to emulate a acrylic window

# changes to index.html
open your main file [index.html] then,
1) wrap your main content in a div tag with some styles as given (feel free to edit and experiment)

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <img class="background">
    <div class="main">
        <!--your content here-->
    </div>
</body>
</html>
```

2) add some css to the background and main
```
body,
html {
    width: 100%;
    height: 100%;
    padding: 0px;
    margin: 0px;
    font-family: sans-serif;
    background-color: #171717;
    overflow: hidden;
}

.background {
    position: absolute;
    width: 100%;
    height: 100%;
    filter: blur(50px);
    opacity: 0.075;
}
.main {
    position: absolute;
    top: 0px;
    left: 0px;
    width: 100vw;
    height: 100vh;
    padding: 0px;
    margin: 0px;
    display: flex;
    flex-direction: column;
    overflow: auto;
}
```
3)  `import` screen from electron then add the following to each of your window 
``` 
    const primaryDisplay = screen.getPrimaryDisplay()
    const { width, height } = primaryDisplay.workAreaSize
    win.on("move", () => {
      win.webContents.send('message', JSON.stringify(win.getPosition()));
    })
    win.on("maximize", () => {
      win.webContents.send('maximized', "");
      win.webContents.send('screenSize', JSON.stringify({ x: width, y: height }));
    })
    win.on("minimize", () => {
      win.webContents.send('message', JSON.stringify(win.getPosition()));
      win.webContents.send('screenSize', JSON.stringify({ x: width, y: height }));
    })
    win.on("focus", () => {
      win.webContents.send('message', JSON.stringify(win.getPosition()));
      win.webContents.send('screenSize', JSON.stringify({ x: width, y: height }));
    })
    setTimeout(() => {
      win.webContents.send('screenSize', JSON.stringify({ x: width, y: height }));
    }, 1000);
```
4) then, add the following to your `preload.js` file
```


window.addEventListener('DOMContentLoaded', () => {
    const testFolder = process.env.APPDATA + "/Microsoft/Windows/Themes/CachedFiles";
    const { readdir, watch, } = require('fs');
    const { ipcRenderer, nativeImage } = require("electron");
    readdir(testFolder, (err, files) => {
      document.body.querySelector("img").src = (nativeImage.createFromPath(process.env.APPDATA + "/Microsoft/Windows/Themes/CachedFiles/" + files[0]).toDataURL());
    });
    watch(testFolder, (eventType, filename) => {
      setTimeout(() => {
        readdir(testFolder, (err, files) => {
          document.body.querySelector("img").src = (nativeImage.createFromPath(process.env.APPDATA + "/Microsoft/Windows/Themes/CachedFiles/" + files[0]).toDataURL());
        });
      }, 10000);
    });
    ipcRenderer.on('message', function (event, text) {
      var pos = JSON.parse(text);
      document.body.querySelector("img").style.left = "-" + Math.abs(pos[0]) + "px";
      document.body.querySelector("img").style.top = "-" + Math.abs(pos[1]) + "px";
    });
    ipcRenderer.on('screenSize', function (event, text) {
      var pos = JSON.parse(text);
      document.body.querySelector("img").style.width = pos.x + "px";
      document.body.querySelector("img").style.height = pos.y + "px";
    });
    ipcRenderer.on('maximized', function (event, text) {
      document.body.querySelector("img").style.left = "0px";
      document.body.querySelector("img").style.right = "0px";
    });
  });
```

You can change the CSS properties i.e. opacity, filter:blur(`px`), and background 
Feel free to give suggestion to make this project better
Thank you
