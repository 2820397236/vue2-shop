<template>
	<div>
		<transition name="router-fade" mode="out-in">
			<keep-alive>
			    <router-view v-if="$route.meta.keepAlive"></router-view>
			</keep-alive>
    	</transition>
    	<transition name="router-fade" mode="out-in">
			<router-view v-if="!$route.meta.keepAlive"></router-view>
		</transition>
    </div>
</template>

<script>

  	export default {
    	
  	}

</script>

<style lang="scss">
  	@import './style/common';
	.router-fade-enter-active, .router-fade-leave-active {
		.main_container{
			transition: opacity .2s;
		}
	  	
	}
	.router-fade-enter, .router-fade-leave-active {
		.main_container{
			opacity: 0;
		}
	}
</style>
