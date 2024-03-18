<template>
  <div class="card mb-3 text-bg-light text-light bg-opacity-25" :style="optionalStyles">
    <div class="card-body">
      <div class="row">
        <div class="col-3">
          <i class="bi overview-icon bigIcon" :class="[icon, color]"></i>
        </div>
        <div class="col-9">
          <h2 class="card-title">{{ amount }} <small class="fs-5" v-if="amountsuffix">{{ amountsuffix }}</small></h2>
          <p class="card-text">{{ title }}</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    title: {},
    icon: {},
    amount: {},
    amountsuffix: {},
    color: {default: "bg-primary"},
    optionalStyles: {}
  }
}
</script>

<style>
.bigIcon{
  font-size: 36px;
}
</style>