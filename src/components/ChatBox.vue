<template>
    <!-- <p>{{ JSON.stringify(symps) }}</p> -->
    <div class="chatbox">
        <div class="chatbox-messages">
            <div v-for="(message, index) in messages" :key="index">
                <div :class="{ 'chatbox-out': message.user == 1, 'chatbox-in': message.user != 1 }">
                    <div class="chatbox-message-text">{{ message.text }}</div>
                    <div class="chatbox-message-timestamp">{{ message.timestamp }}</div>
                </div>
            </div>
            <TypingBub v-if="p_msg == 1" />
        </div>
        <form class="chatbox-form" @submit.prevent="sendMessage">
            <input type="text" v-model="newMessage" placeholder="Type a message...">
            <button type="submit">Send</button>
        </form>
    </div>
</template>
  
<script>
import TypingBub from '@/components/TypingBubble.vue'
export default {
    props: {
        symps: Array
    },
    mounted() {
        this.addMessage(this.getMedBotGreeting(), 0)
    },
    components: {
        TypingBub
    },
    computed: {
        apiUrl() {
            return this.$store.state.api
        }
    },
    data() {
        return {
            messages: [],
            newMessage: '',
            p_msg: 0,
            symptomsMatched: []
        }
    },
    methods: {
        processMessage(msg) {
            this.p_msg = 1

            let symptoms = this.findMatchingKeys(msg, this.symps)

            if (this.isGreeting(msg)) {
                this.addMessage(this.getMedBotGreeting(), 0)
            } else if (this.isDoneWithSymptoms(msg)) {
                this.getAndSetRemedies()
            } else {
                if (symptoms && symptoms.length > 0) {
                    this.symptomsMatched.push(symptoms)
                    this.addMessage(this.getGenericResponse(), 0)
                }
                else {
                    this.addMessage(this.nahiMilaGenericResponse(), 0)
                }
            }

            this.p_msg = 0
        },
        getAndSetRemedies() {

            const paramsArray = this.symptomsMatched
            const apiEndpoint = this.apiUrl + '/chatbot-response/suggest-remedy'

            const paramsString = paramsArray.map(param => encodeURIComponent(param) + '=1').join('&');
            const url = `${apiEndpoint}`;

            console.debug('Symptoms matched : ' + this.symptomsMatched)
            const dat = this.arrayToObjectMap(this.symptomsMatched)
            this.axios.get(url,{ params:dat })
                .then(response => {
                    if (response.data) {
                        const dat = response.data.body.data;
                        this.addMessage(dat.description, 0)
                        const symptomMessage = 'Here are some of the precautions you can take: ' + dat.precaution.toStrin()
                        this.addMessage(symptomMessage, 0)

                        // let mac = this
                        // setTimeout(()=>{
                        //     mac.addMessage('I have res')
                        // },2000)
                    }
                })
                .catch(error => console.error(error));


        },
        arrayToObjectMap(arr) {
            return arr.reduce((obj, item) => {
                obj[item] = 1;
                return obj;
            }, {});
        },
        getMedBotGreeting() {
            const greetings = [
                "Hello! I'm MedBot, here to assist you with any medical symptoms or concerns you may have.",
                "Greetings! My name is MedBot, how can I assist you today?",
                "Welcome to MedBot, I'm here to help you diagnose any symptoms you may be experiencing.",
                "Hi there! My name is MedBot, let's work together to figure out what's going on with your health.",
            ];
            return greetings[Math.floor(Math.random() * greetings.length)];
        },
        findMatchingKeys(str, arr) {
            if (arr && arr.length);
            else
                return []

            const strWords = str.toLowerCase().split(/\W+/).sort().join(' ');
            const matches = [];

            for (const [key, value] of arr[0]) {
                // console.debug('Value to match : ' + value)
                const keyWords = value.toLowerCase().split(/\W+/).sort().join(' ');

                if (strWords.includes(keyWords)) {
                    matches.push(key);
                }
            }

            return matches;
        },
        isDoneWithSymptoms(str) {
            const negatingWords = ["no", "not", "none", "neither", "never", "nothing", "nowhere", "nope", "nada", "nah", "negative", "deny", "refuse", "reject", "stop", "quit", "end", "cancel", "abort", "withdraw", "dismiss", "ignore", "forget", "over", "done", "finished", "complete", "that's all", "that's it"];

            const words = str.toLowerCase().split(" ");
            for (const word of words) {
                if (negatingWords.includes(word)) {
                    return true;
                }
            }

            return false;
        },
        isGreeting(str) {
            const greetings = ['hello', 'hi', 'hey', 'howdy', 'hola', 'good morning', 'good afternoon', 'good evening'];
            const words = str.toLowerCase().split(' ');
            for (let i = 0; i < words.length; i++) {
                if (greetings.includes(words[i])) {
                    return true;
                }
            }
            return false;
        },
        addMessage(msg, user = 1) {
            if (msg.trim() !== '') {
                const now = new Date();
                const timestamp = `${now.getHours()}:${now.getMinutes()}`;
                this.messages.push({
                    text: msg.trim(),
                    timestamp: timestamp,
                    user: user
                });
            }
        },
        nahiMilaGenericResponse() {

            const responses = [
                "I'm sorry, I couldn't find any matching symptoms. Could you describe any other symptoms you're experiencing?",
                "Unfortunately, I couldn't find any symptoms that match what you've described. Can you share any additional symptoms with me?",
                "Hmm, I'm not sure I understand the symptoms you're describing. Can you provide any additional information?",
                "It looks like the symptoms you've provided don't match anything in my database. Can you tell me about any other symptoms you might be experiencing?",
                "I'm not finding any symptoms that match what you've told me. Are there any other symptoms you can share?"
            ];
            const randomIndex = Math.floor(Math.random() * responses.length);
            return responses[randomIndex];


        },
        getGenericResponse() {
            const responses = [
                "Got it. Anything else?",
                "Understood. Do you have any other symptoms?",
                "I see. Anything else you want to tell me?",
                "Thank you. Can you describe any other symptoms you might be experiencing?",
                "Okay. Is there anything else you want to share with me?"
            ];

            const responses2 = [
                "Is there anything else you'd like to add?",
                "Is there anything else that concerns you?",
                "Do you have any other symptoms you'd like to mention?",
                "Is there anything else you want to tell me?",
                "Anything else on your mind?",
                "Is there anything else I can help you with?",
                "What else can you tell me?",
                "Anything else you'd like me to know?",
                "Do you have any other concerns?",
                "What other symptoms are you experiencing?",
                "Can you tell me more about your symptoms?",
                "Anything else I should know?",
                "Is there anything else you'd like to discuss?"
            ];

            const randomIndex = Math.floor(Math.random() * responses.length);
            return responses[randomIndex];
        },
        sendMessage() {
            if (this.newMessage.trim() !== '') {
                const now = new Date();
                const timestamp = `${now.getHours()}:${now.getMinutes()}`;
                this.messages.push({
                    text: this.newMessage.trim(),
                    timestamp: timestamp,
                    user: 1
                });
                const msg = this.newMessage
                this.processMessage(msg)
                this.newMessage = '';

            }
        }
    }
};
</script>
  
<style>
.chatbox {
    position: fixed;
    margin-top: 70px;
    bottom: 0;
    left: 50%;
    transform: translateX(-50%);
    background-color: #f1f1f1;
    border: 1px solid #ccc;
    border-radius: 5px;
    box-shadow: 0px 2px 5px rgba(0, 0, 0, 0.3);
    max-width: 500px;
    width: 100%;
}

.chatbox-messages {
    height: 80vh;
    overflow-y: scroll;
    padding: 10px;
}

.chatbox-in {
    margin-bottom: 10px;
    display: flex;
    flex-direction: column;
    align-items: flex-start;
}

.chatbox-out {
    margin-bottom: 10px;
    display: flex;
    flex-direction: column;
    align-items: flex-end;
}

.chatbox-in>.chatbox-message-text {
    background-color: #454545;
    font-family: 'Roboto Mono', sans-serif;
    color: #87CBB9;
    padding: 5px;
    padding-right: 10px;
    padding-left: 10px;
    border-radius: 5px;
    box-shadow: 0px 2px 5px rgba(0, 0, 0, 0.1);
    max-width: 80%;
}

.chatbox-message-text {
    background-color: #fff;
    padding: 5px;
    padding: 5px;
    padding-right: 10px;
    padding-left: 10px;
    border-radius: 5px;
    box-shadow: 0px 2px 5px rgba(0, 0, 0, 0.1);
    max-width: 80%;
}

.chatbox-message-timestamp {
    font-size: 12px;
    color: #666;
    margin-top: 5px;
}

.chatbox-form {
    display: flex;
    align-items: center;
    padding: 10px;
}

.chatbox-form input[type="text"] {
    flex: 1;
    padding: 10px;
    border-radius: 5px;
    border: none;
    box-shadow: 0px 2px 5px rgba(0, 0, 0, 0.1);
    margin-right: 10px;
}

.chatbox-form button[type="submit"] {
    background-color: #4CAF50;
    color: #fff;
    border: none;
    padding: 10px;
    border-radius: 5px;
    cursor: pointer;
}
</style>
  