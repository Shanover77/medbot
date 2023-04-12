<template>
    <div class="p-3 text-center">
        <h1><i class="fas fa-stethoscope"></i> MedBot</h1>
    </div>

    
    <div v-if="symptoms && symptoms.length>0">
        <ChatBox :symps="symptoms"/>
    </div>
</template>

<script>
import ChatBox from '@/components/ChatBox.vue'
export default {
    components:{
        ChatBox
    },
    data() {
        return {
            symptoms: []
        }
    },
    mounted(){
        this.loadAll()
    },
    methods: {
        loadAll() {
            this.axios.get(this.apiUrl + '/chatbot-response/problems')
                .then(response => {
                    const dat= response.data.body.data

                    const arr = Object.entries(dat)
                    this.symptoms.push(arr)
                })
                .catch(error => {
                    console.error(error);
                });

        }
    },
    computed:{
        apiUrl(){
            return this.$store.state.api
        }
    }
}
</script>

<style></style>