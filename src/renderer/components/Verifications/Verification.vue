<template >
  <!-- TODO: Split this into more files? -->
  <div>
    <nav class="navbar navbar-expand-lg navbar-light bg-light">
      <a class="navbar-brand" href="#">{{ title }}</a>
      <el-button
        type="primary"
        icon="el-icon-plus"
        @click="requestFormActive = true">Request Presentation</el-button>
      <el-button
        type="primary"
        icon="el-icon-refresh"
        @click="$emit('verification-refresh',)"></el-button>
    </nav>
    <el-collapse v-model="expanded_items">
      <ul class="list">
        <el-collapse-item
          v-if="presentations.length"
          v-for="presentation in presentations"
          v-bind:title="presentation.connection_their_label + ' ' + presentation.presentation_exchange_id"
          :name="presentation.presentation_exchange_id"
          :key="presentation.presentation_exchange_id">
          <el-row>
            <div>
              <vue-json-pretty
                :deep=1
                :data="presentation">
              </vue-json-pretty>
            </div>
            <el-button v-on:click="collapse_expanded(presentation)">^</el-button>
          </el-row>
        </el-collapse-item>
      </ul>
    </el-collapse>
    <el-dialog title="Request Presentation" :visible.sync="requestFormActive">
      <el-form :model="requestForm">
        <!-- Connection -->
        <el-form-item label="Connection:" :label-width="formLabelWidth">
          <el-select
            v-model="requestForm.connection_id"
            no-data-text="No active connections"
            filterable
            placeholder="Connection">
            <el-option
              v-for="connection in connections"
              :key="connection.connection_id"
              :label="connection.their_label"
              :value="connection.connection_id">
            </el-option>
          </el-select>
        </el-form-item>

        <el-form-item label="Name:" :label-width="formLabelWidth">
          <el-input
            v-model="requestForm.name"
            placeholder="Proof Request Name"></el-input>
        </el-form-item>

        <!-- Comment -->
        <el-form-item
          label="Comment:"
          :label-width="formLabelWidth">
          <el-input
            v-model="requestForm.comment"
            placeholder="Optional Comment"
            type="textarea"></el-input>
        </el-form-item>

        <!-- Dynamic Attributes -->
        <el-form-item
          v-for="(attribute, index) in requestForm.attributes"
          :label="'Attribute ' + index"
          :label-width="formLabelWidth"
          :key="'attribute_' + index"
          class="additions">
          <el-input
            v-model="requestForm.attributes[index].name"
            placeholder="Attribute name">
            <el-button
              slot="append"
              icon="el-icon-close"
              @click="remove_attribute(index)"></el-button>
          </el-input>
          <el-form-item  label="Restrictions:" class="restrictions">
            <el-select
              :disabled="true"
              v-model="requestForm.attributes[index].restrictions.cred_def"
              filterable
              no-data-text="No credential definitions"
              placeholder="Credential Definition">
              <el-option
                v-for="cred_def in cred_defs"
                :key="cred_def.cred_def_id"
                :label="cred_def.cred_def_id"
                :value="cred_def">
              </el-option>
            </el-select>
            <el-select
              :disabled="true"
              v-model="requestForm.attributes[index].restrictions.trusted_issuer"
              filterable
              no-data-text="No registered trusted issuers"
              placeholder="Trusted Issuers">
              <el-option
                v-for="issuer in trusted_issuers"
                :key="issuer.did"
                :label="issuer.label"
                :value="issuer.did">
              </el-option>
            </el-select>
          </el-form-item>
        </el-form-item>

        <el-divider v-if="requestForm.predicates.length > 0 && requestForm.attributes.length > 0"></el-divider>

        <!-- Dynamic Predicates -->
        <el-form-item
          v-for="(predicate, index) in requestForm.predicates"
          :label="'Predicate ' + index"
          :label-width="formLabelWidth"
          :key="'predicate_' + index"
          class="additions">
          <el-input
            v-model="requestForm.predicates[index].name"
            placeholder="Attribute Name">
            <el-button
              slot="append"
              icon="el-icon-close"
              @click="remove_predicate(index)"></el-button>
          </el-input>
          <el-input
            v-model="requestForm.predicates[index].threshold"
            placeholder="Threshold">
            <el-select
              v-model="requestForm.predicates[index].p_type"
              slot="prepend"
              placeholder="type"
              class="ptype">
              <el-option
                v-for="opt in p_type_options"
                :key="opt"
                :label="opt"
                :value="opt"></el-option>
            </el-select>
          </el-input>
          <el-form-item label="Restrictions:" class="restrictions">
            <el-select
              :disabled="true"
              v-model="requestForm.predicates[index].restrictions.cred_def"
              filterable
              no-data-text="No credential definitions"
              placeholder="Credential Definition">
              <el-option
                v-for="cred_def in cred_defs"
                :key="cred_def.cred_def_id"
                :label="cred_def.cred_def_id"
                :value="cred_def">
              </el-option>
            </el-select>
            <el-select
              :disabled="true"
              v-model="requestForm.predicates[index].restrictions.trusted_issuer"
              filterable
              no-data-text="No registered trusted issuers"
              placeholder="Trusted Issuers">
              <el-option
                v-for="issuer in trusted_issuers"
                :key="issuer.did"
                :label="issuer.label"
                :value="issuer.did">
              </el-option>
            </el-select>
          </el-form-item>
        </el-form-item>

        <el-button
          type="primary"
          icon="el-icon-plus"
          @click="add_attribute">Add attribute</el-button>
        <el-button
          type="primary"
          icon="el-icon-plus"
          @click="add_predicate">Add predicate</el-button>

      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="requestFormActive = false">Cancel</el-button>
        <el-button type="primary" @click="send">Confirm</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<style>
.ptype {
  width: 75px;
}
.additions .el-input {
  margin-bottom: .5em;
}
.restrictions {
  margin-top: .5em;
}
.restrictions .el-select {
  margin-bottom: .5em;
}
</style>

<script>
import VueJsonPretty from 'vue-json-pretty';

export default {
  name: 'verification',
  props: [
    'title',
    'list',
    'editable',
    'connections',
    'cred_defs',
    'trusted_issuers'
  ],
  components: {
    VueJsonPretty,
  },
  data () {
    return {
      expanded_items:[],
      requestFormActive: false,
      requestForm: {
        // This differs subtly from presentation proposal
        connection_id: '', // Who
        comment: '', // Optional comment
        name: '',
        attributes: [
          // Key: (can be whatever) property name,
          // {
          //   "name": "attribute_name"
          //   "restrictions": {
          //     "cred_def_id": "asdf", etc.
          //   }
          // }

          /*
          {
            "name": "attribute_name",
            "cred_def_id":"",
            "mime_type":"",
            "value": ""
          }
          */
        ],
        predicates: [
          // Key: (can be whatever) property name,
          // {
          //   "name": "attribute_name",
          //   "p_type": ">=",
          //   "p_value": 18,
          //   "restrictions": {
          //     "cred_def_id": "asdf", etc.
          //   }
          // }
        ],
        presentation_predicates:[
          /**
           *{
           *  "name": "",
           *  "cred_def_id": "",
           *  "predicate": "",
           *  "threshold": "",
           *}
           */
        ]
      },
      formLabelWidth: '100px',
      p_type_options: ['>', '<', '>=', '<=']
    }
  },
  computed: {
    connection_map: function() {
      let map =  this.connections.reduce((acc, item) => {
        acc[item.connection_id] = item;
        return acc;
      }, {});
      console.log(map);
      return map;
    },
    presentations: function() {
        let joinedPresentationsConnections = this.list.map((item) => {
          var index = this.connections.map(function(x) {return x.connection_id; }).indexOf(item.connection_id);
          var objectFound = this.connections[index];
          if (index >= 0) {
            item.connection_their_label = this.connections[index].their_label
          }else{
            item.connection_their_label = "deleted connetion"
          }
        return item;
        },{})
        return joinedPresentationsConnections;
      },
    completed_verifications: function() {
      return this.list.filter(pres_exch => pres_exch.state === 'verified');
    }
  },
  methods: {
    collapse_expanded: function(creddef) {
      this.expanded_items = this.expanded_items.filter(
        item => item != creddef.presentation_exchange_id
      );
    },
    send: function() {
      let values = {
        connection_id: this.requestForm.connection_id,
        name: this.requestForm.name,
        comment: this.requestForm.comment,
        attributes: this.requestForm.attributes,
        predicates: this.requestForm.predicates
      }
      this.requestFormActive = false;

      this.requestForm.connection_id = '';
      this.requestForm.name = '';
      this.requestForm.comment = '';
      this.requestForm.attributes = [];
      this.requestForm.predicates = [];

      this.$emit('presentation-request', values);
    },
    add_attribute: function() {
      this.requestForm.attributes.push({
        name: '',
        restrictions: {
          cred_def: undefined,
          trusted_issuer: undefined
        },
      });
    },
    remove_attribute: function(index) {
      this.requestForm.attributes.splice(index, 1);
    },
    add_predicate: function() {
      this.requestForm.predicates.push({
        name: '',
        p_type: '',
        restrictions: {
          cred_def: undefined,
          trusted_issuer: undefined,
        },
      });
    },
    remove_predicate: function(index) {
      this.requestForm.predicates.splice(index, 1);
    }
  }
}
</script>
