<template>
  <div>
    <v-snackbar v-bind="$attrs" v-model="snackbar.show" :key="snackbar.key" :class="'v-snackbars v-snackbars-'+idx+'-'+identifier" :timeout="-1" v-for="(snackbar,idx) in snackbars">
      {{ snackbar.message }}
      <template v-slot:action>
        <slot name="action" :close="removeMessage" :id="snackbar.key" :index="idx" :message="snackbar.message">
          <v-btn text @click="removeMessage(snackbar.key)">close</v-btn>
        </slot>
      </template>
    </v-snackbar>
    <css-style>
      .v-snackbars .v-snack__wrapper {
        transition: {{topOrBottom}} 500ms;
        {{topOrBottom}}: 0;
      }
    </css-style>
    <css-style :key="idx" v-for="idx in len">
      .v-snackbars.v-snackbars-{{idx}}-{{identifier}} > .v-snack__wrapper {
        {{topOrBottom}}:{{ idx*distance }}px;
      }
    </css-style>
  </div>
</template>

<script>
export default {
  name:'v-snackbars',
  props:{
    'messages':{
      type:Array,
      default:() => []
    },
    'timeout':{
      type:[Number,String],
      default:5000
    },
    'distance':{
      type:[Number,String],
      default:55
    }
  },
  data () {
    return {
      topOrBottom:'bottom',
      len:0, // we need it to have a css transition
      snackbars:[], // array of {key, message, show(true)}
      identifier:Date.now()+((Math.random()+"").slice(2))
    }
  },
  components:{
    'css-style': {
      render: function (createElement) {
        return createElement('style', this.$slots.default)
      }
    }
  },
  watch:{
    messages () {
      this.setSnackbars();
    }
  },
  methods:{
    setSnackbars () {
      // update the text if it changes
      for (let i=0; i<this.snackbars.length && i<this.messages.length; i++) {
        // if the text is blank, then remove the notification
        if (this.messages[i]==="") {
          this.removeMessage(this.snackbars[i].key)
          return;
        }
        this.snackbars[i].message = this.messages[i];
      }
      for (let i=this.snackbars.length; i<this.messages.length; i++) {
        let key = i+'-'+Date.now();
        this.snackbars.push({
          key:key,
          message:this.messages[i],
          show:true
        })
        if (this.timeout > 0) {
          setTimeout(() => this.removeMessage(key), this.timeout*1);
        }
      }
      if (this.snackbars.length > this.len) this.len=this.snackbars.length;
    },
    removeMessage (key) {
      let idx = this.snackbars.findIndex(s => s.key === key);
      if (idx > -1) {
        this.snackbars.splice(idx, 1);
        this.$emit('update:messages', this.snackbars.map(o => o.message));
      }
    }
  },
  created () {
    if (typeof this.$attrs.top !== "undefined" && this.$attrs.top !== false) this.topOrBottom='top';
    this.setSnackbars();
  }
}
</script>
