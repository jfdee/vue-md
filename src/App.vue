<template>
  <div class="main">
    <div ref="display" class="main__input" style="border: 1px solid black" />
    <textarea ref="input" v-model="value" class="main__input" />
  </div>
</template>

<script>
  export default {
    name: 'App',
    data() {
      return {
        value: '',
        htmlValue: '',
        key17: false, // ctrl
        key66: false, // b
        key73: false, // i
        handleKeys: [17, 66, 73]
      }
    },
    computed: {
      inputElement() {
        return this.$refs.input
      },
      displayElement() {
        return this.$refs.display
      }
    },
    watch: {
      value() {
        this.setHtmlValue()
      },
      htmlValue(val) {
        this.displayElement.innerHTML = val
      },
      key66(pressed) {
        if (!pressed) return
        if (!this.key17) return
        this.value = this.getMarkdownValue('**')
      },
      key73(pressed) {
        if (!pressed) return
        if (!this.key17) return
        this.value = this.getMarkdownValue('*')
      },
    },
    mounted() {
      window.addEventListener('keydown', this.handleKey)
      window.addEventListener('keyup', this.handleKey)
    },
    beforeUnmount() {
      window.removeEventListener('keydown', this.handleKey)
      window.removeEventListener('keyup', this.handleKey)
    },
    methods: {
      handleKey(e) {
        if (!this.handleKeys.includes(e.keyCode)) return
        const pressed = e.type === 'keydown'
        if (e.keyCode === 17) this.key17 = pressed
        if (e.keyCode === 66) this.key66 = pressed
        if (e.keyCode === 73) this.key73 = pressed
      },
      setHtmlValue() {
        let html = this.value.replaceAll('\n', '<br>')
        html = this.getHtmlValue(html, '**', 'strong')
        html = this.getHtmlValue(html, '*', 'em')
        this.htmlValue = html
      },
      getMarkdownValue(to) {
        const value = this.value
        const start = this.inputElement.selectionStart
        const end = this.inputElement.selectionEnd

        const before = value.slice(0, start)
        const selected = value.slice(start, end)
        const selectedNoSpaces = selected.trim()
        if (selectedNoSpaces === '') return value

        const selectedNoSpacesLeft = selected.trimStart()
        const spacesLeft = selected.replace(selectedNoSpacesLeft, '')
        const selectedNoSpacesRight = selected.trimEnd()
        const spacesRight = selected.replace(selectedNoSpacesRight, '')

        const after = value.slice(end)
        return before + spacesLeft + to + selectedNoSpaces + to + spacesRight + after
      },
      getHtmlValue(value, from, to) {
        /*
          * 1. "asd" -> ['asd'] -> "asd"
          *
          * 2. "**asd** asd" -> ['', 'asd', 'asd'] -> ['asd'] -> "<b>asd</b> asd"
          * 3. "asd asd **asd**" -> ['asd asd ', 'asd', ''] -> ['asd asd '] -> "asd asd <b>asd</b>"
          *
          * 4. "asd **asd** asd" -> ['asd ', 'asd', ' asd'] -> "asd <b>asd</b> asd"
          * 5. "asd **asd asd** asd" -> ['asd ', 'asd asd', ' asd'] -> "asd <b>asd asd</b> asd"
          * 6. "asd **asd** **asd** asd"
          *
          * 7. "asd*asd" -> "asd*asd"
          * 8. "asd*asd*asd*" -> "asd*asd<i>asd</i>"
          * 9. "*a*" -> ['', 'a', ''] -> "<i>a</i>"
          *
        */
        const openTag = '<' + to + '>'
        const closeTag = '</' + to + '>'
        const parts = value.split(from)
        console.log(parts)
        if (parts.length === 1) return value // 1.
        if (parts.length === 2) return value // 7.

        // prepare start and end
        if (parts[0] === '' && parts[parts.length - 1] === '') return openTag + parts[1] + closeTag // 9.

        // prepare start part
        let start = ''
        let startRemoved = false
        const firstIndex = 0
        if (parts[firstIndex] === '') {  // 2.
          const secondIndex = 1
          start = openTag + parts[secondIndex] + closeTag
          parts.splice(secondIndex, 1)
          parts.splice(firstIndex, 1)
          startRemoved = true
        } else {
          start = parts[firstIndex]
          parts.splice(firstIndex, 1)
        }
        //

        // prepare end part
        let end = ''
        const lastIndex = parts.length - 1
        if (parts[lastIndex] === '') { // 3.
          const preLastIndex = lastIndex - 1
          end = openTag + parts[preLastIndex] + closeTag
          parts.splice(preLastIndex, 1)
          parts.splice(lastIndex, 1)
        } else {
          end = parts[lastIndex]
          parts.splice(lastIndex, 1)  // 2. 4. 5.
        }
        //
        if (parts.length === 2) return start + parts.join('') + end // 8.

        // combine middle part
        let middle = ''
        let skip = startRemoved // skip current part
        for (let item of parts) {
          if (item.trim() === '') { // '   ' = '** **'
            middle += item
            skip = !skip
            continue
          }
          middle += skip ? item : `${openTag}${item}${closeTag}`
          skip = !skip
        }
        //

        return start + middle + end
      },
    },
  }
</script>

<style scoped>
  .main {
    width: 600px;
    height: 600px;
    margin: 10% auto;
    box-shadow: 1px 1px 1px 1px gray;
    padding-top: 1px;
  }
  .main__input {
    margin: 25px;
    width: 542px;
    height: 200px;
  }
  .main > textarea {
    overflow: auto;
    outline: none;

    -webkit-box-shadow: none;
    -moz-box-shadow: none;
    box-shadow: none;

    resize: none; /*remove the resize handle on the bottom right*/

    -webkit-user-select  : text;
  }
</style>