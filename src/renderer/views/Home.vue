<template>
  <div>
    <section class="hero is-dark is-bold">
      <div class="hero-body">
        <div class="content ">
          <h1 class="title is-4 has-text-centered is-centered">PEM Checker</h1>


          <div class="file has-name">
            <label class="file-label">
              <input class="file-input" type="file" @change="previewFiles" name="resume" />
              <span class="file-cta">
                <span class="file-icon">
                  <i class="fas fa-upload"></i>
                </span>
                <span class="file-label">Choose a file…</span>
              </span>
              <span class="file-name">{{fileName}}</span>
            </label>
          </div>

          <template v-if="notBefore">
            
            <div class="pem-dates">
              <h6 class="is-size-6">Dates</h6>
            <p>Not Before: {{ notBefore }}</p>
            <p>Not After: {{ notAfter }}</p>
          </div>
          </template>

          <div class="attribute-body" v-if="pemdetails.length">
            <div class="attribute-subject" v-for="(heading, index) in pemdetails" :key="index">
              <h6 class="is-size-6">{{heading.title}}</h6>
              <ul>
                <li v-for="(attributes, index) in heading.Attributes" :key="index">
                  {{attributes.title}}: {{attributes.value}}
                </li>
              </ul>
            </div>
          </div>


        </div>
      </div>
    </section>
  </div>
</template>

<script>
import Vue from 'vue'
const fs = require('fs')
let pki = require('node-forge').pki

export default Vue.extend({
  name: 'Home',
  data() {
    return {
      searchValue: '',
      notBefore:null,
      notAfter:null,
      pemdetails:[],
      fileName: ''
    }
  },
  methods: {
    previewFiles(event) {
      const filepath = event.target.files[0].path
      
      this.fileName = event.target.files[0].name;
      

      this.pemdetails = [];

      try {
         var ca = pki.certificateFromPem(fs.readFileSync(filepath, 'binary'))
      } catch {
       
       let newFile = '';
       let newFileArray = [];

        const dataArray = fs.readFileSync(filepath, 'binary').split("\n");
        
        let collectLine = false;

        // for (let index = 0; index < array.length; index++) {
        //   const element = array[index];
          
        // }
        dataArray.forEach(line => {
          if (line.startsWith('----')) {
            collectLine = !collectLine;
            if (!collectLine) {
              newFile += line + "\n"
            }
          }
          if (collectLine) {
            newFile += line + "\n"
          }
        });

        // We have stripped everything now we just need to move the rsa
        const indexOfRSAStart = newFile.indexOf("-----BEGIN RSA PRIVATE KEY-----");
        const indexOfRSAEnd = newFile.indexOf("-----END RSA PRIVATE KEY-----") + 29;
        const lengthOfString = newFile.length;

        const valueToMove = newFile.substring(indexOfRSAStart,indexOfRSAEnd);
        newFile = newFile.slice(indexOfRSAEnd,lengthOfString) + valueToMove;

        var ca = pki.certificateFromPem(newFile);

      }
      this.notBefore = ca.validity.notBefore;
      this.notAfter = ca.validity.notAfter;

      if (ca.subject.attributes) {
        const subject = [];
        ca.subject.attributes.forEach(el => {
          subject.push({title:this.createTitle(el.name), value: el.value});
        });
        // this.pemdetails.subject = subject;
        this.pemdetails.push({title:'Subject', Attributes: subject})
      }

      if (ca.issuer.attributes) {
        const issuer = [];
        ca.issuer.attributes.forEach(el => {
          issuer.push({title:this.createTitle(el.name), value: el.value});
        });
        this.pemdetails.push({title:'Issuer', Attributes: issuer})
      }

    },
    createTitle(titleToConvert){
      var text = titleToConvert ? titleToConvert : 'ID';
      var result = text.replace( /([A-Z])/g, " $1" );
      return result.charAt(0).toUpperCase() + result.slice(1);
    },
  },
})
</script>

<style>
.hero-body {
  height: 100vh;
}
.file {
  justify-content: center;
}
.attribute-body .is-size-6 {
  color: #fff;
}
.attribute-subject {
  margin-bottom: 10px;
}
.pem-dates .is-size-6 {
  color: #fff;
}
.content .pem-dates p {
  margin-bottom: 0;
}
.pem-dates {
  margin-bottom: 15px;
}
.file {
  margin-bottom: 15px;
}

</style>
