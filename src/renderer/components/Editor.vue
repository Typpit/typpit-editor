<template>
	<div id="container" style="height:500px;border:1px solid #ccc"></div>
</template>

<script>
  /* global amdRequire */
  var app = require('electron').remote.app
  var path = require('path')
  var Y = require('yjs')
  require('y-websockets-client')(Y)
  require('y-memory')(Y)
  require('y-array')(Y)
  require('y-text')(Y)

  function loadMonacoEditor () {
    function uriFromPath (_path) {
      var pathName = path.resolve(_path).replace(/\\/g, '/')
      if (pathName.length > 0 && pathName.charAt(0) !== '/') {
        pathName = '/' + pathName
      }
      return encodeURI('file://' + pathName)
    }

    amdRequire.config({
      baseUrl: uriFromPath(path.join(app.getAppPath(), './node_modules/monaco-editor/dev'))
    })

    // workaround monaco-css not understanding the environment
    self.module = undefined

    // workaround monaco-typescript not understanding the environment
    self.process.browser = true

    amdRequire(['vs/editor/editor.main'], function () {
      var that = this
      Y({
        db: {
          name: 'memory' // use memory database adapter.
          // name: 'indexeddb' // use indexeddb database adapter instead for offline apps
        },
        connector: {
          name: 'websockets-client',
          room: 'my-room', // clients connecting to the same room share data
          url: 'http://localhost:1234'
        },
        sourceDir: null,
        share: {
          monaco: 'Text' // y.share.textarea is of type y-text
        }
      }).then(function (y) {
        window.yMonaco = y
        // The Yjs instance `y` is available
        // y.share.* contains the shared types

        var editor = that.monaco.editor.create(document.getElementById('container'), {
          language: 'javascript'
        })

        // Bind `y.share.monaco`
        y.share.monaco.bindMonaco(editor)
      })
    })
  }
  export default {
    name: 'editor',
    components: { },
    methods: {
      open (link) {
        this.$electron.shell.openExternal(link)
      }
    },
    mounted () {
      loadMonacoEditor()
    }
  }
</script>

<style>
  @import url('https://fonts.googleapis.com/css?family=Source+Sans+Pro');

  * {
    box-sizing: border-box;
    margin: 0;
    padding: 0;
  }

  body { font-family: 'Source Sans Pro', sans-serif; }

  #wrapper {
    background:
      radial-gradient(
        ellipse at top left,
        rgba(255, 255, 255, 1) 40%,
        rgba(229, 229, 229, .9) 100%
      );
    height: 100vh;
    padding: 60px 80px;
    width: 100vw;
  }

  #logo {
    height: auto;
    margin-bottom: 20px;
    width: 420px;
  }

  main {
    display: flex;
    justify-content: space-between;
  }

  main > div { flex-basis: 50%; }

  .left-side {
    display: flex;
    flex-direction: column;
  }

  .welcome {
    color: #555;
    font-size: 23px;
    margin-bottom: 10px;
  }

  .title {
    color: #2c3e50;
    font-size: 20px;
    font-weight: bold;
    margin-bottom: 6px;
  }

  .title.alt {
    font-size: 18px;
    margin-bottom: 10px;
  }

  .doc p {
    color: black;
    margin-bottom: 10px;
  }

  .doc button {
    font-size: .8em;
    cursor: pointer;
    outline: none;
    padding: 0.75em 2em;
    border-radius: 2em;
    display: inline-block;
    color: #fff;
    background-color: #4fc08d;
    transition: all 0.15s ease;
    box-sizing: border-box;
    border: 1px solid #4fc08d;
  }

  .doc button.alt {
    color: #42b983;
    background-color: transparent;
  }
</style>
