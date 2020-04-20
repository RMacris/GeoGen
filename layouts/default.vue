<template>
  <div id="Main">
    <nuxt />
    <footer>
      <div></div>
      <div>
        Github:
        <a href="https://github.com/RMacris" target="_blank">Rafael Macris</a>
      </div>
    </footer>
  </div>
</template>

<style>
a {
  color: rgb(200, 200, 200);
  text-decoration: none;
  text-emphasis: none;
}
a:hover {
  color: #eee;
  text-decoration: underline;
}

body {
  background-color: rgb(50, 50, 50);
}
footer {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  height: 15%;
  background-color: rgb(35, 35, 35);
  box-shadow: 0 -4px 4px 0px rgba(20, 20, 20, 0.8);
  border-top: 1px solid rgb(15, 15, 15);
}
footer > * {
  color: rgb(200, 200, 200);
}
#Main {
  display: flex;
  flex-direction: column;

  height: 100vh;
  max-height: 100vh;
}
html {
  font-family: 'Source Sans Pro', -apple-system, BlinkMacSystemFont, 'Segoe UI',
    Roboto, 'Helvetica Neue', Arial, sans-serif;
  font-size: 16px;
  word-spacing: 1px;
  -ms-text-size-adjust: 100%;
  -webkit-text-size-adjust: 100%;
  -moz-osx-font-smoothing: grayscale;
  -webkit-font-smoothing: antialiased;
  box-sizing: border-box;
}

*,
*:before,
*:after {
  box-sizing: border-box;
  margin: 0;
}

.button--green {
  display: inline-block;
  border-radius: 4px;
  border: 1px solid #3b8070;
  color: #3b8070;
  text-decoration: none;
  padding: 10px 30px;
}

.button--green:hover {
  color: #fff;
  background-color: #3b8070;
}

.button--grey {
  display: inline-block;
  border-radius: 4px;
  border: 1px solid #35495e;
  color: #35495e;
  text-decoration: none;
  padding: 10px 30px;
  margin-left: 15px;
}

.button--grey:hover {
  color: #fff;
  background-color: #35495e;
}
</style>
