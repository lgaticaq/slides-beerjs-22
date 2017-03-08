title: Beerjs #18
author:
  name: Leonardo Gatica
  twitter: lgaticaq
  url: https://about.me/lgatica
  email: lgatica@protonmail.com
output: index.html
theme: juanbrujo/cleaver-beerjs
style: style.css
controls: true

--

<h1>Como hacer test para <div id="huemul"><img src="https://avatars0.githubusercontent.com/u/17724906?v=3&s=64"></img></div></h1>

--

# hubot-test-helper

Es un _mock_ de hubot para simplificar el testing de tus scripts

```shell
yarn add -D hubot-test-helper
```

--

```coffeescript
expect = require("chai").expect
Helper = require("hubot-test-helper")
helper = new Helper("../src/echo.coffee")

describe "example", ->
  beforeEach ->
    @room = helper.createRoom(httpd: false)
  afterEach ->
    @room.destroy()
  context "hubot echo", ->
    beforeEach (done) ->
      @room.user.say("user", "hubot echo hello world")
      setTimeout(done, 100)
    it "should get a hello world", ->
      expect(@room.messages).to.eql([
        ["user", "hubot echo hello world"],
        ["hubot", "hello world"]
      ])
```

--

```coffeescript
expect = require("chai").expect
Helper = require("hubot-test-helper")
helper = new Helper("../src/echo.coffee")

describe "example", ->
  beforeEach ->
    @room = helper.createRoom(httpd: false)
  afterEach ->
    @room.destroy()
  context "private message", ->
    beforeEach (done) ->
      @room.user.say("user", "hubot tell me a secret")
      setTimeout(done, 100)
    it "should get a hello world", ->
      expect(@room.privateMessages).to.eql({
        "user": [["hubot", "top secret"]]
      })
```

--

```coffeescript
expect = require("chai").expect
http = require("http")
Helper = require("hubot-test-helper")
helper = new Helper("../src/http.coffee")

process.env.EXPRESS_PORT = 8080

describe "httpd-world", ->
  beforeEach ->
    @room = helper.createRoom()
  afterEach ->
    @room.destroy()
  context "GET /hello/world", ->
    beforeEach (done) ->
      http.get "http://localhost:8080/hello/world", (@response) => done()
      .on "error", done
    it "responds with status 200", ->
      expect(@response.statusCode).to.equal 200
```

--

```coffeescript
querystring = require("querystring")
http = require("http")
expect = require("chai").expect
Helper = require("hubot-test-helper")
helper = new Helper("../src/http.coffee")

process.env.EXPRESS_PORT = 8080

describe "httpd-world", ->
  beforeEach ->
    @room = helper.createRoom()
  afterEach ->
    @room.destroy()
  context "POST /hello/world", ->
    beforeEach (done) ->
      data = querystring.stringify(foo: "bar")
      req = http.request
        hostname: "localhost"
        port: 8080
        path: "/hello/world"
        method: "POST"
        headers:
          "Content-Type": "application/x-www-form-urlencoded"
          "Content-Length": Buffer.byteLength(data)
      , (response) =>
        @response = response
      req.write(data)
      req.end(done)
    it "responds with status 200", ->
      expect(@response.statusCode).to.equal 200
```

--

# nock

Libreria para crear mock de api externas

```shell
yarn add -D nock
```

--

```coffeescript
expect = require("chai").expect
Helper = require("hubot-test-helper")
nock = require("nock")
helper = new Helper("../src/echo.coffee")

describe "example", ->
  beforeEach ->
    @room = helper.createRoom(httpd: false)
    nock.disableNetConnect()
  afterEach ->
    @room.destroy()
    nock.cleanAll()
  context "hubot echo", ->
    beforeEach (done) ->
      nock("http://yourapi").post("/some").query(foo: "bar").reply 200,
        foo: "bar"
      @room.user.say("user", "hubot echo hello world")
      setTimeout(done, 100)
    it "should get a hello world", ->
      expect(@room.messages).to.eql([
        ["user", "hubot echo hello world"],
        ["hubot", "hello world"]
      ])
```

--

```coffeescript
expect = require("chai").expect
Helper = require("hubot-test-helper")
nock = require("nock")
helper = new Helper("../src/echo.coffee")

describe "example", ->
  beforeEach ->
    @room = helper.createRoom(httpd: false)
    nock.disableNetConnect()
  afterEach ->
    @room.destroy()
    nock.cleanAll()
  context "hubot echo", ->
    beforeEach (done) ->
      nock("http://yourapi").post("/some").query(foo: "bar")
        .replyWithError("Server error")
      @room.user.say("user", "hubot echo hello world")
      setTimeout(done, 100)
    it "should get a hello world", ->
      expect(@room.messages).to.eql([
        ["user", "hubot echo hello world"],
        ["hubot", "hello world"]
      ])
```

--

```coffeescript
expect = require("chai").expect
Helper = require("hubot-test-helper")
nock = require("nock")
helper = new Helper("../src/echo.coffee")

describe "example", ->
  beforeEach ->
    @room = helper.createRoom(httpd: false)
    nock.disableNetConnect()
  afterEach ->
    @room.destroy()
    nock.cleanAll()
  context "hubot echo", ->
    beforeEach (done) ->
      nock("http://yourapi").post("/some").query(foo: "bar")
        .replyWithError("Server error")
      @room.user.say("user", "hubot echo hello world")
      setTimeout(done, 100)
    it "should get a hello world", ->
      expect(@room.messages).to.eql([
        ["user", "hubot echo hello world"],
        ["hubot", "hello world"]
      ])
```
--

# proxyquire

Libreria para sobrescribir el comportamiento de librerias externas

```bash
yarn add -D proxyquire
```

--

```coffeescript
expect = require("chai").expect
Helper = require("hubot-test-helper")
proxyquire = require("proxyquire")

myStub =
  getData: (data) ->
    return new Promise (resolve, reject) ->
      if data is "1"
        resolve(foo: "bar")
      else
        reject(new Error("Not found"))
proxyquire("../src/script.coffee", {"my-lib": myStub})
helper = new Helper("../src/script.coffee")

describe "example", ->
  beforeEach ->
    @room = helper.createRoom(httpd: false)
  afterEach ->
    @room.destroy()
  context "hubot echo", ->
    beforeEach (done) ->
      @room.user.say("user", "hubot echo hello world")
      setTimeout(done, 100)
    it "should get a hello world", ->
      expect(@room.messages).to.eql([
        ["user", "hubot echo hello world"],
        ["hubot", "hello world"]
      ])
```

--

# Simular estar usando Slack en hubot

--

```coffeescript
expect = require("chai").expect
Helper = require("hubot-test-helper")
helper = new Helper("../src/script.coffee")

describe "example", ->
  beforeEach (done) ->
    @room = helper.createRoom(httpd: false)
    @room.robot.adapter.client =
      web:
        chat:
          postMessage: (channel, text, options) =>
            @postMessage =
              channel: channel
              text: text
              options: options
            done()
  afterEach ->
    @room.destroy()
  context "hubot echo", ->
    beforeEach (done) ->
      @room.user.say("user", "hubot echo hello world")
      setTimeout(done, 100)
    it "should get a hello world", ->
      expect(@room.messages).to.eql([
        ["user", "hubot echo hello world"],
        ["hubot", "hello world"]
      ])
```

--

```coffeescript
expect = require("chai").expect
Helper = require("hubot-test-helper")
helper = new Helper("../src/script.coffee")

describe "example", ->
  beforeEach (done) ->
    @room = helper.createRoom(httpd: false)
  afterEach ->
    @room.destroy()
  context "hubot echo", ->
    beforeEach (done) ->
      @room.robot.brain.data._private["foo"] = "bar"
      @room.user.say("user", "hubot echo hello world")
      setTimeout(done, 100)
    it "should get a hello world", ->
      expect(@room.messages).to.eql([
        ["user", "hubot echo hello world"],
        ["hubot", "hello world"]
      ])
```

--

```coffeescript
expect = require("chai").expect
Helper = require("hubot-test-helper")
helper = new Helper("../src/script.coffee")

describe "example", ->
  beforeEach (done) ->
    @room = helper.createRoom(httpd: false)
    @room.robot.adapter.client =
      web:
        users:
          list: () ->
            return new Promise (resolve) ->
              resolve(members: [{id: 1, name: "user"}])
    @room.robot.brain.userForId "user",
      name: "user"
      id: 1
  afterEach ->
    @room.destroy()
  context "hubot echo", ->
    beforeEach (done) ->
      @room.user.say("user", "hubot echo hello world")
      setTimeout(done, 100)
    it "should get a hello world", ->
      expect(@room.messages).to.eql([
        ["user", "hubot echo hello world"],
        ["hubot", "hello world"]
      ])
```

--

# Crear scripts compatibles con Slack y Shell

```coffeescript
module.exports = (robot) ->
  robot.router.post "/travis-ci/:room", (req, res) ->
    channel = req.params.room
    if robot.adapter.constructor.name in ["SlackBot", "Room"]
      robot.adapter.client.web.chat.postMessage(
        "##{channel}", null, attachments)
      res.send("success")
    else
      robot.messageRoom(channel, text)
      res.send("success")
```
