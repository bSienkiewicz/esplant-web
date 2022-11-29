<template>
  <div class="__site">
    <NavBar />
    <div class="content">
      <div class="header">
        <img
          src="~/static/logo.png"
          id="logo-main"
          alt=""
          style="filter: brightness(60%)"
        />
      </div>
      <div class="main">
        <div class="main__left">
          <div class="main__left__top">
            <h1
              style="
                font-weight: 500;
                color: white;
                text-shadow: 2px 2px 6px rgba(66, 68, 90, 0.68);
                border-bottom: 1px solid white;
              "
            >
              TWÓJ OGRÓD
            </h1>
          </div>
          <div class="main__left__bottom">
            <nuxt-link to="/plant/1" class="plant">
              <h2>Fuksja</h2>
              <div class="child plant__image--1"></div>
            </nuxt-link>
            <nuxt-link to="/plant/2" class="plant">
              <h2>Kaktus</h2>
              <div class="child plant__image--2"></div>
            </nuxt-link>
            <nuxt-link to="/plant/3" class="plant">
              <h2>Kwiat testowy</h2>
              <div class="child plant__image--3"></div>
            </nuxt-link>
          </div>
        </div>
        <div class="main__right">
          <div class="main__right__top">
            <h1
              style="
                font-weight: 500;
                color: black;
                border-bottom: 1px solid black;
              "
            >
              SZCZEGÓŁY
            </h1>
          </div>
          <div class="main__right__bottom">
            <div class="details">
              <p>Ostatni odczyt: 16.11.2022 godz. 01.14</p>
              <p>Ostatnie podlanie: 16.11.2022 godz. 01.14</p>
              <p>Temperatura: 20°C</p>
              <p>Pojemność wody: 40%</p>
              <p>Stan czujnika: OK</p>
            </div>

            <div class="water__level">
              <h3 style="z-index: 2">40%</h3>
              <div class="water__bar"></div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'IndexPage',
}
</script>

<style lang="scss">
@import url('https://fonts.googleapis.com/css2?family=Kanit:wght@300;400;500;600&display=swap');
html {
  font-family: 'Kanit', sans-serif;
}

body {
  margin: 0;
  padding: 0;
  background-color: #e3f0e9;
}

* {
  box-sizing: border-box;
}

.__site {
  margin: 0;
  width: 100vw;
  height: 100vh;
}

.content {
  padding: 80px 30px 30px 30px;
  display: grid;
  grid-template-columns: 1fr;
  grid-template-rows: 80px 1fr;
  grid-template-areas:
    'header'
    'main';
  height: 100%;
  max-width: 100%;
  @media screen and (max-width: 768px) {
    padding: 80px 10px 10px 10px;
  }
}

#logo-main {
  max-width: 400px;
  height: auto;
  display: block;
  @media screen and (max-width: 768px) {
    max-width: 300px;
  }
}

.main {
  // background: #ccd9d2;
  border-radius: 5px;
  display: grid;
  grid-template-columns: 1fr 1fr;
  max-height: 100%;
  overflow: auto;
  padding: 20px;

  @media screen and (max-width: 768px) {
    grid-template-columns: 1fr;
    padding: 10px;
  }
  .main__left {
    background: #457342;
    color: black;
    border-radius: 5px;
    -webkit-box-shadow: inset 0px 0px 24px 0px rgba(0, 0, 0, 0.114);
    -moz-box-shadow: inset 0px 0px 24px 0px rgba(0, 0, 0, 0.114);
    box-shadow: inset 0px 0px 24px 0px rgba(0, 0, 0, 0.286);
  }

  .main__left,
  .main__right {
    display: grid;
    max-height: inherit;
    grid-template-columns: 1fr;
    grid-template-rows: 100px 1fr;
    grid-template-areas:
      'top'
      'bottom';
  }
  .main__left__top,
  .main__right__top {
    display: flex;
    max-height: 100%;
    padding: 0 40px;
    align-items: center;
    justify-content: flex-start;
  }

  .main__left__bottom {
    display: inline-grid;
    grid-template-columns: 1fr 1fr;
    @media screen and (max-width: 1008px) {
      grid-template-columns: 1fr;
    }
    max-height: inherit;
    overflow: auto;
    padding: 0 40px 40px 40px;
    gap: 20px;
    width: 100%;
    justify-content: center;
    .plant {
      height: 250px;
      cursor: pointer;
      width: 100%;
      overflow: hidden;
      border-radius: 5px;
      position: relative;

      &:hover .child,
      &:focus .child {
        transform: scale(1.05);
      }

      .child {
        height: 100%;
        background-size: cover;
        background-position: center;
        transition: all 0.2s ease;

        &:before {
          content: '';
          position: absolute;
          // gradient from transparent to black
          background: linear-gradient(
            180deg,
            rgba(15, 19, 34, 0) 0%,
            rgba(15, 19, 34, 0) 50%,
            rgb(6, 8, 14) 100%
          );
          top: 0;
          left: 0;
          width: 100%;
          height: 100%;
        }
      }

      h2 {
        position: absolute;
        bottom: 20px;
        left: 20px;
        color: white;
        margin: 0;
        z-index: 10;
      }
    }
  }
  .main__right__bottom {
    display: grid;
    grid-template-columns: 1fr 100px;
    grid-template-rows: 1fr;
    padding: 0 40px 40px 40px;
    h3 {
      font-weight: 500;
      color: black;
      margin: 0;
    }
    .water__level {
      height: 100%;
      width: 50px;
      background-color: white;
      border: 1px solid #cbcbcb;
      border-radius: 10px;
      position: relative;
      display: flex;
      justify-content: center;
      align-items: center;
      .water__bar {
        position: absolute;
        height: 40%;
        width: 100%;
        bottom: 0;
        background-color: #9fcbe3;
        border-radius: 4px;
        transition: all 0.2s ease;
      }
    }
  }
}
.plant__image--1 {
  background: url(https://images.unsplash.com/photo-1537039557108-4a42c334fd5e?ixlib=rb-4.0.3&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=735&q=80);
}

.plant__image--2 {
  background: url(https://images.unsplash.com/photo-1463320898484-cdee8141c787?ixlib=rb-4.0.3&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=1170&q=80);
}
.plant__image--3 {
  background: url(https://images.unsplash.com/photo-1509223197845-458d87318791?ixlib=rb-4.0.3&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=949&q=80);
}
</style>
