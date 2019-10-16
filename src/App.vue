<template>
  <div id="app">
    <div class="header">
      Enter GraphQL Endpoint
      <input type="text" v-model="endpoint" />
      <button @click="getSchema">Get Schema</button>
    </div>
    <div class="ooux" v-if="endpointData">
      <div class="ooux-column" v-for="object in filteredData.__schema.types" :key="object.name">
        <div class="ooux-object">{{object.name}}</div>
        <template v-if="object.fields">
          <div
            :class="[
            field.description && `ooux-type--${field.description}`,
            field.type.ofType && `ooux-type--isobject`,
             'ooux-content']"
            v-for="field in object.fields"
            :key="field.name"
          >{{field.name}}</div>
        </template>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
const typesToHide = [
  "Query",
  "Int",
  "String",
  "Boolean",
  "Float",
  "__Schema",
  "__Type",
  "__Field",
  "__TypeKind",
  "__InputValue",
  "__EnumValue",
  "__Directive",
  "__DirectiveLocation",
  "CacheControlScope",
  "Upload"
];

export default {
  name: "app",
  components: {},
  data() {
    return {
      typesToHide: typesToHide,
      endpointData: "",
      endpoint: ""
    };
  },
  computed: {
    filteredData: function() {
      if (this.endpointData) {
        const input = this.endpointData;
        const { __schema } = input;
        console.log("hilfe", typesToHide);

        const isNotOnNaughtyList = ({ name }) =>
          !this.typesToHide.includes(name);

        const types = __schema.types.filter(isNotOnNaughtyList);
        console.log("filtered types", types);
        const output = {
          __schema: {
            types
          }
        };
        return output;
      }
    }
  },
  mounted() {
    if (localStorage.endpoint) {
      this.endpoint = localStorage.endpoint;
      this.getSchema();
    }
  },
  watch: {
    endpoint(newEndpoint) {
      localStorage.endpoint = this.endpoint;
    }
  },

  methods: {
    writeToLocalStorage(e) {},
    async getSchema() {
      try {
        console.log("endpoint:", this.endpoint);
        const res = await axios.post(this.endpoint, {
          query: `
          {
  __schema {
    types {
      name
      fields{
        description
        name
        type{
          ofType{
            name
          }
        }
      }
    }
  }
}`
        });
        this.endpointData = res.data.data;
      } catch (e) {
        console.log("err", e);
      }
    }
  }
};
</script>

<style lang="scss">
body {
  padding: 0;
  margin: 0;
}
.ooux {
  margin-top: 120px;
  --blue: #0176ba;
  --yellow: #f9bb00;
  --red: #ee230d;
  font-size: 12px;
  padding: 20px;
  grid-column-gap: 10px;
  display: grid;
  grid-auto-columns: 1fr;
  grid-auto-flow: column;
}
.ooux-column {
  display: flex;
  flex-direction: column;
}
.ooux-object {
  color: white;
  width: 80px;
  height: 80px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: var(--blue);
  margin-bottom: 10px;
}

.ooux-content {
  width: 80px;
  display: flex;
  align-items: center;
  justify-content: center;
  height: 80px;
  color: black;
  background: var(--yellow);
  margin-bottom: 10px;

  &.ooux-type--isobject {
    color: white;
    background: var(--blue);
  }
}

#app {
  width: 100%;
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
.header {
  position: fixed;
  right: 0;
  left: 0;
  top: 0;
  background: black;
  color: white;
  padding: 20px;
  display: grid;
  grid-auto-flow: column;
  grid-gap: 20px;
  grid-template-rows: 1fr;
  justify-content: flex-start;
  align-content: stretch;
  align-items: center;
  input {
    font: inherit;
    width: 20em;
  }
  button {
    height: 100%;
    font: inherit;
    color: white;
    border: none;
    padding: 0 30px;
    background: fuchsia;
  }
}

/* types */

.ooux-type--OOUX-Metadata {
  background: red;
  color: white;
}
</style>
