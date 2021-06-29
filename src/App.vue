<template>
<div>
  <h1><span class="header-span">p</span>roject<span class="header-span">Ygg</span>drasil</h1>
  <p id="subtitle">A Textploration Game</p>
</div>
  <Console></Console>
</template>

<script>
import Console from './components/Console.vue'

export default {
  name: 'App',
  components: {
    Console
  }
}
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Montserrat:ital,wght@0,200;0,400;0,900;1,200;1,400;1,900&display=swap');
@import url('https://fonts.googleapis.com/css2?family=Montserrat:ital,wght@0,100;0,200;0,300;0,400;0,500;0,600;0,700;0,800;0,900;1,100;1,200;1,300;1,400;1,500;1,600;1,700;1,800;1,900&display=swap');
@import url('https://fonts.googleapis.com/css2?family=Fira+Code:wght@400;500;600;700&display=swap');

body {
    background-color: rgb(41, 41, 41);
}

#header-div {
    display: block;
    margin-left: auto;
    margin-right: auto;
    margin-top: 0;
    margin-bottom: 0;
    padding: 0;
    width: 75%;
}

h1 {
    font-family: monospace;
    font-weight: 300;
    font-style: italic;
    font-size: 5em;
    text-align: center;
    color: whitesmoke;
    margin: .25em auto .2em;
    ;
}
.header-span {
    color: rgb(255, 227, 68);
}
#subtitle {
    font-family: "Fira Code", monospace;
    font-weight: 300;
    font-style: normal;
    font-size: 1em;
    text-align: center;
    color: whitesmoke;
    margin: 0 auto 2em;
}
</style>
