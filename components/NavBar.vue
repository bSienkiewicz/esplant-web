<template>
  <nav>
    <img src="~/static/logo-sm.png" id="logo-sm" alt="" />
    <div class="nav-items">
      <NuxtLink to="/" class="nav-item"> Panel </NuxtLink>
      <a href="/" class="nav-item"> Szczegóły </a>
    </div>
  </nav>
</template>

<script>
export default {}
</script>

<style lang="scss" scoped>
nav {
  background-color: #e3f0e9;
  position: fixed;
  top: 0;
  width: 100%;
  height: 60px;
  -webkit-box-shadow: inset 0px 0px 24px 0px rgba(0, 0, 0, 0.114);
  -moz-box-shadow: inset 0px 0px 24px 0px rgba(0, 0, 0, 0.114);
  box-shadow: 0px 0px 24px 0px rgba(0, 0, 0, 0.286);
}

#logo-sm {
  position: absolute;
  top: 10px;
  left: 10px;
  width: 40px;
  height: 40px;
}

.nav-items {
  display: flex;
  justify-content: flex-start;
  gap: 30px;
  align-items: center;
  height: 100%;
  margin: 0 auto;
  padding: 0 50px;
  .nav-item {
    padding: 8px 20px;
    color: black;
    position: relative;
    text-decoration: none;
  }
}

a.nuxt-link-exact-active {
  &:after {
    content: '';
    position: absolute;
    bottom: 0;
    left: 0;
    right: 0;
    width: 50%;
    height: 2px;
    background-color: black;
    margin: auto;
  }
}
</style>
