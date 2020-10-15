<template>
  <q-page class="flex flex-center">
    <div class="container container-fluid">
      <div class="form-group">
        <label for="exampleFormControlTextarea1">Enter lyrics:</label>
        <textarea class="form-control" id="exampleFormControlTextarea1" rows="3" v-model="lyrics"></textarea>
    </div>
    <div class="form-group">
      <label for="exampleFormControlInput1">Number of Omitting Words: </label>
      <input type="number" class="form-control" id="exampleFormControlInput1" v-model="omitNumber">
    </div>
    <button type="button" @click="startAction()" class="btn btn-primary">Start!</button>
  <div>
    <xmp>
      <span v-html="formatted"></span>
    </xmp>
  </div>
  <div>
    <button type="button" @click="endAction()" class="btn btn-primary">End!</button>
  </div>
    </div>
  </q-page>
</template>

<script>
export default {
  name: 'PageIndex',
  data () {
    return {
      lyrics: null,
      omitNumber: null,
      formatted: null,
      result: null
    }
  },
  methods: {
    endAction () {
      for (var i = 0; i < this.result.length; i++) {
        if (this.result[i] === document.getElementById(`${i}`).value) {
          document.getElementById(`${i}`).style.borderColor = 'green'
        } else {
          document.getElementById(`${i}`).style.borderColor = 'red'
        }
      }
    },
    startAction () {
      if (!this.lyrics) {
        alert('Textarea is empty!')
      } else {
        var r = /(\w|\s)*\w(?=")|\w+('\w+)?/gm
        var array = this.lyrics.match(r)
        var copyArray = []
        array.forEach(word => {
          if (word.length > 3) {
            copyArray.push(word)
          }
        })
        console.log(copyArray)
        var result = this.getRandom(copyArray, this.omitNumber)
        var lyrics = this.lyrics
        var index = 0
        result.forEach(word => {
          lyrics = lyrics.replace(word, `<input type="text" class="rounded" style="width: ${word.length * 10}px;" v-model="words[${index}]" id="${index}">`)
          index += 1
        })
        this.formatted = lyrics
        console.log(result)
        this.result = result
        console.log(lyrics)
      }
    },
    getRandom (arr, n) {
      var result = new Array(n),
        len = arr.length,
        taken = new Array(len)
      if (n > len) {
        throw new RangeError('getRandom: more elements taken than available')
      }
      while (n--) {
        var x = Math.floor(Math.random() * len)
        result[n] = arr[x in taken ? taken[x] : x]
        taken[x] = --len in taken ? taken[len] : len
      }
      return result
    }
  }
}
</script>
