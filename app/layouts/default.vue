<template>
  <div>
    <b-navbar class="navbar navbar-dark bg-primary">
      <b-navbar-brand to="/">Component Catalog</b-navbar-brand>
      <b-navbar-nav>
        <b-nav-item to="/all-components">Components</b-nav-item>
        <b-nav-item to="/all-teams">Teams</b-nav-item>
        <b-nav-item to="/all-areas">Areas</b-nav-item>
        <b-nav-item to="/all-tests">Tests</b-nav-item>
        <b-nav-item :to="{ name: 'all-scanners' }">Scanners</b-nav-item>
      </b-navbar-nav>
    </b-navbar>
    <div class="mx-1 my-2">
      <Message />
    </div>
    <div id="content" class="mx-3">
      <Nuxt />
    </div>
  </div>
</template>

<style>
#content a {
  color: #3498db;
  text-decoration: underline;
}

#content a.btn {
  color: white;
  text-decoration: none;
}

#content .table-primary a {
  color: white;
}

#content table {
  margin-bottom: 1rem;
}
</style>
