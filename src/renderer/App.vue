<template>
  <div id="app">
    <!-- <landing-page></landing-page> -->
    <dashboard-page></dashboard-page>
  </div>
</template>

<script>
import LandingPage from "@/components/LandingPage";
import DashboardPage from "@/components/DashboardPage";

export default {
  name: "project-server",
  components: {
    LandingPage,
    DashboardPage
  }
};
</script>

<style>
/* CSS */
</style>
