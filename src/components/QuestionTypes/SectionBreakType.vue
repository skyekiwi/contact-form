<template>
  <div class="f-content" v-if="question.content">
    <span class="f-section-text">{{ question.content }}</span>
  </div>
</template>

<script>
  import BaseType from './BaseType.vue'
  import { QuestionType } from '../../models/QuestionModel'

  export default {
    extends: BaseType,
    name: QuestionType.SectionBreak,
    methods: {
      onEnter() {
        this.dirty = true

        this._onEnter()
      },

      isValid() {
        return true
      }
    }
  }
</script>
