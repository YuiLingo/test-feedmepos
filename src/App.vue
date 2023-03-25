<script setup>
import Dashboard from './components/Dashboard.vue'
</script>

<template>
  	<nav class="navbar navbar-expand-lg bg-light">
  		<div class="container-fluid">
  			<a class="navbar-brand" href="javascripts:void(0)"><img src="./assets/bootstrap-logo.svg" alt="Logo" width="30" height="24" class="d-inline-block align-text-top"> Home</a>
  			<button class="navbar-toggler" type="button" data-bs-toggle="collapse"
  				data-bs-target="#navbarSupportedContent" aria-controls="navbarSupportedContent" aria-expanded="false"
  				aria-label="Toggle navigation">
  				<span class="navbar-toggler-icon"></span>
  			</button>
  		</div>
  	</nav>
  	<div class="container-fluid mt-3">
      <Dashboard />
    </div>
</template>