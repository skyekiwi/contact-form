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
            <span class="fh2">Thank you. 🙏</span>
            <span class="f-section-text">
              You can review your answers or press submit. Follow us on the social links as well!
            </span>
          </p>
          <p class="f-description">Note: your data will only be accessible by the SkyeKiwi Team.</p>
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
  import QuestionModel, { QuestionType, ChoiceOption } from './models/QuestionModel'
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
            title: 'Stay Tuned @ SkyeKiwi',
            content: '',
            description: 'Powered by a (very) simplified version of SkyeKiwi Protocol. Your submit is decyrptable by the SkyeKiwi Team only and we will keep it a secret for ya. 🍿️',
            type: QuestionType.SectionBreak
          }),
          new QuestionModel({
            id: 'full_name',
            tagline: 'Hi! We\'d like to know you! 😊',
            title: 'Name',
            type: QuestionType.Text,
            required: true,
            placeholder: 'Your Name'
          }),
          new QuestionModel({
            id: 'email',
            tagline: "Nice to meet you 👀",
            title: 'What\'s a good email address?',
            description: "No Spam. We Promise.",
            type: QuestionType.Email,
            required: true,
            placeholder: 'Your Email...'
          }),
          new QuestionModel({
            id: 'types',
            title: 'Which of the following choice(s) best describes you?',
            type: QuestionType.MultipleChoice,
            multiple: true,
            helpText: 'Select all that apply. 👇',
            required: true,
            options: [
              new ChoiceOption({
                label: 'Developer'
              }),
              new ChoiceOption({
                label: 'Program Manager'
              }),
              new ChoiceOption({
                label: 'Designer'
              }),
              new ChoiceOption({
                label: 'Investor'
              }),
              new ChoiceOption({
                label: 'Other'
              }),
              new ChoiceOption({
                label: 'Secret'
              })
            ]
          }),
          new QuestionModel({
            id: 'channel',
            title: 'How did you hear about us?',
            type: QuestionType.MultipleChoice,
            helpTextShow: false,
            required: true,
            multiple: true,
            options: [
              new ChoiceOption({
                label: 'Web3 Foundation'
              }),
              new ChoiceOption({
                label: 'Website'
              }),
              new ChoiceOption({
                label: 'Medium'
              }),
              new ChoiceOption({
                label: 'Twitter'
              }),
              new ChoiceOption({
                label: 'Word of Mouth'
              }),
              new ChoiceOption({
                label: 'Others'
              })
            ],
            jump: {
              _other: '_submit'
            }
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
          types: data.answers[3],
          channel: data.answers[4],
        }))

        const encrypted = AsymmetricEncryption.encrypt(
          privateKey, msg, Util.hexToU8a('5ca2c480ced265f9fca9ebf7a423bdf4143bb1e634301b1069ceae53267c4f10')
        )

        fetch('https://formapi.skye.kiwi/contact', {
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
