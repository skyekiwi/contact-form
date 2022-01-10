<template>
  <div>
    <header class="vff-header">
      <div class="f-container">
        <svg width="250px"  viewBox="0 0 4377 893" version="1.1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink">
            <title>SkyeKiwi</title>
            <g id="Page-1" stroke="none" stroke-width="1" fill="none" fill-rule="evenodd">
                <text id="𝚺𝜎-Copy" font-family="STIXGeneral-Regular, STIXGeneral" font-size="500" font-weight="normal" fill="#FFFFFF">
                    <tspan x="180" y="610">𝚺𝜎</tspan>
                </text>
                <circle id="Oval-Copy" stroke="#FFFFFF" stroke-width="30" cx="447" cy="447" r="411"></circle>
                <text id="Skye·Kiwi-Copy" font-family="Courier" font-size="600" font-weight="normal" fill="#FFFFFF">
                    <tspan x="1112" y="629">Skye·Kiwi</tspan>
                </text>
            </g>
        </svg>
      </div>
    </header>

    <flow-form
      ref="flowform"
      v-on:complete="onComplete"
      v-on:submit="onSubmit"
      v-bind:questions="questions"
      v-bind:language="language"
      v-bind:standalone="true"
    >
     <template v-slot:complete>
        <div class="f-section-wrap">
          <p>
            <span class="fh2">Sit tight ... </span>
            <span class="f-section-text">
              Follow us on social media. 
            </span>
          </p>
          <p class="f-description"> </p>
        </div>  
      </template>
      <template v-slot:completeButton>
        <div class="f-submit" v-if="!submitted">
          <button 
            class="o-btn-action"
            ref="button"
            type="submit"
            href="#"
            v-on:click.prevent="onSendData()"
            aria-label="Press to submit"
          >
            <span>{{ language.submitText }}</span>
          </button>
          <a class="f-enter-desc"
            href="#"
            v-on:click.prevent="onSendData()"
            v-html="language.formatString(language.pressEnter)">
          </a>
        </div>

        <p class="text-success" v-if="submitted">Submitted succesfully.</p>
      </template>
    </flow-form>
  </div>
</template>

<script>
  import FlowForm from './components/FlowForm.vue'
  import QuestionModel, { QuestionType } from './models/QuestionModel'
  import LanguageModel from './models/LanguageModel'
  import { randomBytes } from 'tweetnacl'
  import { Util, AsymmetricEncryption } from '@skyekiwi/protocol'

  export default {
    name: 'skyekiwi',
    components: {
      FlowForm
    },
    data() {
      return {
        submitted: false,
        completed: false,
        language: new LanguageModel(),
        questions: [
          new QuestionModel({
            id: 'intro',
            title: 'Get excited! KiwiSign is coming soon ...',
            content: 'Subscribe to be the first to know about our launch.',
            description: 'Powered by the SkyeKiwi Protocol. Your submit is decyrptable by the SkyeKiwi Team only. 🍿️',
            type: QuestionType.SectionBreak
          }),
          new QuestionModel({
            id: 'full_name',
            title: 'Name',
            type: QuestionType.Text,
            required: true,
            placeholder: 'Your Name'
          }),
          new QuestionModel({
            id: 'email',
            title: 'Email',
            description: "No Spam. We Promise.",
            type: QuestionType.Email,
            required: true,
            placeholder: 'Your Email...'
          })
        ]
      }
    },
    mounted() {
      document.addEventListener('keyup', this.onKeyListener)
    },
    beforeDestroy() {
      document.removeEventListener('keyup', this.onKeyListener)
    },
    methods: {
      onKeyListener($event) {
        // We've overriden the default "complete" slot so
        // we need to implement the "keyup" listener manually.

        if ($event.key === 'Enter' && this.completed && !this.submitted) {
          this.onSendData()
        }
      },

      /* eslint-disable-next-line no-unused-vars */
      onComplete(completed, questionList) {
        // This method is called whenever the "completed" status is changed.
        this.completed = completed
      },

      /* eslint-disable-next-line no-unused-vars */
      onSubmit(questionList) {
        // This method will only be called if you don't override the
        // completeButton slot.
        this.onSendData()
      },
      
      async onSendData() {
        // Set `submitted` to true so the form knows not to allow back/forward
        // navigation anymore.
        this.$refs.flowform.submitted = true

        this.submitted = true

        /* eslint-disable-next-line no-unused-vars */
        const data = this.getData()

        const privateKey = randomBytes(32)
        const publicKey = AsymmetricEncryption.getPublicKey(privateKey)
        const msg = Util.stringToU8a(JSON.stringify({
          full_name: data.answers[1],
          email: data.answers[2],
        }))

        const encrypted = AsymmetricEncryption.encrypt(
          privateKey, 
          msg, 
          // a public Curve25519 key of SkyeKiwi 
          Util.hexToU8a('5ca2c480ced265f9fca9ebf7a423bdf4143bb1e634301b1069ceae53267c4f10')
        )

        fetch('https://formapi.skye.kiwi/contact/kiwisign', {
          method: 'POST',
          mode: 'cors',
          headers: {
            'Content-Type': 'application/json'
          },
          body: JSON.stringify({
            pubkey: Util.u8aToHex(publicKey),
            msg: Util.u8aToHex(encrypted)
          })
        }).then(() => {
          window.location.href = "https://skye.kiwi"
        })
      },

      getData() {
        const data = {
          questions: [],
          answers: []
        }

        this.questions.forEach(question => {
          if (question.title) {
            let answer = question.answer
            if (Array.isArray(answer)) {
              answer = answer.join(', ')
            }

            data.questions.push(question.id)
            data.answers.push(answer)
          }
        })

        return data
      }
    },
  }
</script>

<style lang="css">
@import '~@ditdot-dev/vue-flow-form/dist/vue-flow-form.css';
@import './assets/css/themes/theme-minimal.css';
</style>
