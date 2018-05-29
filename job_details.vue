<template>
    <div> <!-- without an outer container div this component template will not render -->
        <loader v-if="!dataLoaded"></loader>
        <transition name="fade">
            <div v-if="dataLoaded" v-cloak>
        		<div class="site_container">
        		    <div class="margin_40"></div>
				    <div class="job_details_container">
				        <div class="clearfix">
    					    <div class="promo_img" v-lazy:background-image="currentJob.image_url"></div>
    					    <div class="promo_content">
    					        <h3 class="">{{ currentJob.store_name }}</h3>
    					        <p v-if="currentJob.store_category">{{ currentJob.store_category }}</p>
    							<p>{{ currentJob.category }}</p>
    							<hr>
    					        <p class="job_position" v-if="locale=='en-ca'">{{ $t("jobs_page.position") }}: {{currentJob.name }}</p>
    					        <p class="job_position" v-else>{{ $t("jobs_page.position") }}: {{ currentJob.name_2 }}</p>
    					        <p v-if="currentJob.job_type" class="job_position">{{ $t("jobs_page.job_type") }}: {{ checkJobType(currentJob) }}</p>
    							<p class="job_date">{{ $t("jobs_page.end_date") }}: {{ currentJob.end_date | moment("MMMM DD, YYYY", timezone)}}</p>
    					    </div>
    					</div>
					    <div class="job_details_desc">
					        <p class="job_desc_title">{{ $t("jobs_page.desc_title") }}:</p>
        				    <div v-if="locale=='en-ca'" v-html="currentJob.rich_description"></div>
        				    <div v-else v-html="currentJob.rich_description_2"></div>
        				    <a v-if="currentJob.website" :href="'//' + currentJob.website" target="_blank">
        				        <div class="details_store_website">{{ $t("stores_page.store_website") }}</div>
        				    </a>
        				</div>
					</div>
                </div>
            </div>
        </transition>
	</div>
</template>

<script>
    define(['Vue', 'vuex', 'moment', 'vue-lazy-load'], function(Vue, Vuex, moment, VueLazyload) {
        Vue.use(VueLazyload);
        return Vue.component("job-details-component", {
            template: template, // the variable template will be injected,
            data: function() {
                return {
                    dataLoaded: false,
                    currentJob: null,
                    storeJobs : null
                }
            },
            props:['id', 'locale'],
            beforeRouteUpdate(to, from, next) {
                this.currentJob = this.findJobBySlug(to.params.id);
                    if (this.currentJob === null || this.currentJob === undefined){
                        this.$router.replace({ name: '404'});
                    }
                next();
            },
            created(){
                this.loadData().then(response => {
                    this.dataLoaded = true;
                    this.updateCurrentJob(this.id);
                });
            },
            watch: {
                currentJob : function (){
                    var property_name = this.property.name
                    if(this.currentJob != null) {
                        if (this.currentJob.store != null && this.currentJob.store != undefined){
                            if(_.includes(this.currentJob.store.image_url, 'missing')) {
                                this.currentJob.image_url = "https://via.placeholder.com/400x400/757575";
                            } else {
                                this.currentJob.image_url = this.currentJob.store.store_front_url_abs
                            }
                            this.currentJob.store_name = this.currentJob.store.name;
                            // this.currentJob.category = 
                        } else if (this.currentJob.store == null || this.currentJob.store == undefined) {
                            this.currentJob.store = {};
                            this.currentJob.image_url =  "https://via.placeholder.com/400x400/757575";
                            this.currentJob.store_name = property_name;
                        }
                    }
                }
            },
            computed: {
                ...Vuex.mapGetters([
                    'property',
                    'timezone',
                    'findJobBySlug'
                ])
            },
            methods: {
                updateCurrentJob (id) {
                    this.currentJob = this.findJobBySlug(id);
                    if (this.currentJob === null || this.currentJob === undefined){
                        this.$router.replace({ name: '404'});
                    }
                },
                loadData: async function() {
                    try {
                        // avoid making LOAD_META_DATA call for now as it will cause the entire Promise.all to fail since no meta data is set up.
                        let results = await Promise.all([this.$store.dispatch("getData", "jobs")]);
                    } catch (e) {
                        console.log("Error loading data: " + e.message);
                    }
                },
                checkJobType(promo) {
                    if(this.locale != "en-ca") {
                        if(promo.job_type == "Full Time"){
                            return "Plein Temps"
                        } else if (promo.job_type == "Part Time"){
                            return "Mi Temps"
                        } else if (promo.job_type == "Full Time/Part Time"){
                            return "Plein/Mi Temps"
                        } else if (promo.job_type == "Seasonal"){
                            return "Travail Saisonnier"
                        }
                    } else {
                        return promo.job_type
                    }
                }
            }
        });
    });
</script>