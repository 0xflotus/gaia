<template>
  <div>
    <div class="columns is-multiline">
      <template v-for="(arg, index) in args">
        <div class="column is-one-third" :key="index">
          <div class="content-article params-div">
            <h2 v-if="arg.type !== 'boolean'">{{ arg.desc }}</h2>
            <p class="control" style="padding-bottom: 5px;" v-if="arg.type === 'textfield'">
              <input class="input is-medium input-bar" v-focus v-model="arg.value" :placeholder="arg.key">
            </p>
            <p class="control" v-else-if="arg.type === 'textarea'">
              <textarea class="textarea input-bar" v-model="arg.value" :placeholder="arg.key"></textarea>
            </p>
            <p class="control" v-else-if="arg.type === 'boolean'">
              <label class="checkbox">
                <input type="checkbox">
                {{ arg.desc }}
              </label>
            </p>
          </div>
        </div>
      </template>
    </div>
    <a class="button is-primary" v-on:click="startPipeline" style="margin-top: 5px;">
      <span class="icon">
        <i class="fa fa-play-circle"></i>
      </span>
      <span>Start Pipeline</span>
    </a>
  </div>
</template>

<script>
export default {
  data () {
    return {
      args: [],
      pipelineID: null
    }
  },

  mounted () {
    // Fetch data from backend
    this.fetchData()
  },

  watch: {
    '$route': 'fetchData'
  },

  methods: {
    fetchData () {
      // look up url parameters
      var pipelineID = this.$route.query.pipelineid
      if (!pipelineID) {
        return
      }
      this.pipelineID = pipelineID

      // reset args
      this.args = []

      this.$http
        .get('/api/v1/pipeline/' + pipelineID, { showProgressBar: false })
        .then(response => {
          if (response.data) {
            let pipeline = response.data

            // Get all arguments
            for (let x = 0, y = pipeline.jobs.length; x < y; x++) {
              let args = pipeline.jobs[x].args
              // we skip vault cause this is automatically filled by the vault.
              if (args) {
                // iterate all arguments
                for (let argID = 0, argTotal = args.length; argID < argTotal; argID++) {
                  // we skip vault arguments cause they are autofilled in the backend.
                  if (args[argID].type !== 'vault') {
                    this.args.push(args[argID])
                  }
                }
              }
            }
          }
        })
        .catch((error) => {
          this.$onError(error)
        })
    },

    startPipeline () {
      // Send start request
      this.$http
        .post('/api/v1/pipeline/' + this.pipelineID + '/start', this.args)
        .then(response => {
          if (response.data) {
            this.$router.push({path: '/pipeline/detail', query: { pipelineid: this.pipelineID, runid: response.data.id }})
          }
        })
        .catch((error) => {
          this.$onError(error)
        })
    }
  }
}
</script>

<style lang="scss">

.params-div {
  padding: 15px;
  width: 100%;
}

.checkbox:hover {
  color: #4da2fc;
}

</style>