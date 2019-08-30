# bili-live-connector

This is a vue project built by vue-cli3.0, served as an example to connect to any Bilibili live room in browser if you want. And the basic functions such as danmaku, gift displaying, welcome message, global broadcast and popularity report has been realized. You can see the detailed message either by inspecting the console or just seeing the printed message in the static page (index.html).

The static page can function well in very early browsers after thousand of testing, I am pretty confident that there's no need to worry about compatibility, =v=. Anyway, if there were any broblem while using this stuff, just post an issue and I'll handle it.

The connection is based on websocket, and most part of its techniques are coming from  [Bili-Live-API](https://github.com/lovelyyoshino/Bilibili-Live-API). To be honest, I just draw a dipper with a gourd as a model, orz.

But this is one of the part that I use to realize my customized live plugins for Bilibili ups. I cannot give the whole source of the project because of the appointment, this part, however, is the core part of all my tricks, I'd like to share it with you all~ I wonder if I am the first one to make an OBS plugin in that way, haha~

The inititialization is quite simple, just follow the instructions below: 

## Project setup

```
npm install
```

### Compiles and hot-reloads for development
```
npm run serve
```

### Compiles and minifies for production
```
npm run build
```

### Run your tests
```
npm run test
```

### Lints and fixes files
```
npm run lint
```

### Customize configuration
See [Configuration Reference](https://cli.vuejs.org/config/).