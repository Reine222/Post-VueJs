# Post-VueJs
Vue JS
# lien en vuejs :
           :href="``+`/actualites/details-news/`+`${ item.slug }`+``"
# lien tres utile 
      https://stackoverflow.com/questions/58695433/django-form-in-vuejs-component-template

# Post de Formulaire Avec VueJs :

# Dans le models.py : 


      class Contact(models.Model):
          name = models.CharField(max_length=250)
          email = models.EmailField(max_length=254)
          sujet = models.CharField(max_length=250)
          message = models.TextField()
          date_add = models.DateTimeField(auto_now=False, auto_now_add=True)
          date_upd = models.DateTimeField(auto_now=True, auto_now_add=False)


# Dans le views.py :


      def contact(request):
          return render(request, 'pages/contact.html')

      def sendContact(request):
          postdata = json.loads(request.body.decode('utf-8'))
          name = postdata['nom']
          email = postdata['email']
          sujet = postdata['sujet']
          message = postdata['message']
          succes = False
          try:
              contact = Contact()
              contact.nom = name
              contact.email = email
              contact.sujet = sujet
              contact.message = message
              contact.save()
              succes = True
              reponse = 'Votre message a bien été enregisté '
          except:
              succes = False
              reponse = "Un probleme survennu lors de l'enregistrement"

          datas = {
              'succes':succes,
              'reponse':reponse,
          }

          return JsonResponse(datas, safe=False)
          
          
# Dans le Template : 

      #<form action="#" class="bg-white p-5 contact-form">
                    <div class="form-group">
                      <input type="text" v-model= "name" class="form-control" placeholder="Your Name">
                    </div>
                    <div class="form-group">
                      <input type="text" v-model= "email" class="form-control" placeholder="Your Email">
                    </div>
                    <div class="form-group">
                      <input type="text" v-model= "sujet" class="form-control" placeholder="Subject">
                    </div>
                    <div class="form-group">
                      <textarea name="" v-model= "message"  cols="30" rows="7" class="form-control" placeholder="Message"></textarea>
                    </div>
                    <div class="form-group">
                      <input type="submit" v-on:click="sendContact" value="Send Message" class="btn btn-primary py-3 px-5">
                    </div>
        #</form>




      <script>
            var ma_vue = new Vue({
                  el: '#myComment',
                  // mounted: {
                  //     ,
                  // },
                  data: {
                      
                      name: '',
                      email: '',
                      suject: '',
                      message: '',
                      isregister: false,
                      result: {'succes': false,'reponse':'' },
                  },
                  delimiters: ["${", "}"],
                  methods: {
                     
                      },
                      sendregister: function () {
                          if (this.verifSaisi(this.name, this.email, this.suject, this.message)){
                              axios.defaults.xsrfCookieName = 'csrftoken'
                              axios.defaults.xsrfHeaderName = 'X-CSRFToken'
                              axios.post('http://127.0.0.1:8000/comment/', {
                              name: this.name,
                              email:  this.email,
                              suject:this.suject,
                              message:this.message,
                              article_id: this.article_id,
                                  }).then(response => {
                                      console.log(response)
                                      this.codesend = true
                                      this.result= response.data
                                      this.name = ''
                                      this.email = ''
                                      this.suject = ''
                                      this.message = ''
                                  })
                                  .catch((err) => {
                                      console.log(err)
                                      console.log(this.article_id)
                                      this.codesend = true
                                      this.result['reponse'] = "Probleme de connexion , veuillez contactez l'administrateur"
                                      this.result['succes'] = false


                              })
                          }

                      }
                  }
              })


          </script>
# POST VU JS
# dans la viexs
      from django.shortcuts import render, redirect
      from django.views.decorators.csrf import csrf_exempt
      from django.http import JsonResponse
      import json

      @csrf_exempt
      def registerProf(request):
          succes = False
          message = ""

          if request.method == 'POST':
              email = request.POST.get('email')
              password = request.POST.get('password')
              prenoms = request.POST.get('prenoms')
              nom = request.POST.get('nom')
              prof = True if request.POST.get('prof') == 'True' else False
              print(email)
              print("*************$", prof)
          try:
              if email and  password is not None and not password.isspace() and prenoms is not None and not prenoms.isspace() and nom is not None and not nom.isspace() :

                  usersmail = User.objects.filter(email=email).count()
                  if usersmail < 1 :
                      validate_email(email)
                      s = User(
                          username=usernamegenerate(nom,prenoms),
                          first_name=prenoms,
                          last_name=nom,
                          email=email
                      )
                      s.save()

                      s.password=password
                      s.set_password(s.password)
                      s.save()
                      profi = s.profile
                      profi.is_prof = prof
                      profi.save()
                      page = render_to_string('mailing/register.html', {
                      'nom':nom,
                      'prenom': prenoms,
                      })
                      subject = 'Inscription MyProf'
                      messagesender.sendmail("My prof","myprofci@gmail.com",email,subject,page)
                      try:
                          us = authenticate(username=s.username, password=password)
                          if us.is_active:
                              login(request, us)
                      except:
                          pass
                      succes =True
                      message = "Connexion Réussite"

                  else:
                      succes =False
                      message = "email existe deja"
              else:
                  succes = False
                  message = "Entrez des données correctes"

          except Exception as e:
              print(e)
              succes = False
              message = "Un probleme survenu, réessayez SVP"

          datas = {
              'succes':succes,
              'message': message,
              'is_prof':prof
          }
          return JsonResponse(datas, safe=False)
 # dans le template
 
      <script src='https://cdn.jsdelivr.net/npm/vue/dist/vue.js'></script>
      <script src='https://unpkg.com/axios/dist/axios.min.js'></script>
      <script src="https://cdn.jsdelivr.net/npm/sweetalert2@9"></script>

      <script>
          var app = new Vue({
              el: "#inscripProf2",
              data: {
                  isregister: false,
                  nom:"",
                  prenoms: "",
                  email: "",
                  password: "",
                  prof: 'True',
                  base_url: window.location.protocol + "//" + window.location.host,
                  url_post: "{% url 'post_registerProf' %}",
                  accueil: "{% url 'connexion' %}",
                  annonce: "{% url 'annonce1' %}"
              },
              delimiters: ["${", "}"],
              methods: {
                  registerProf: function(){
                      if (!this.isregister) {
                          this.error = false;
                          this.isSuccess = false;
                          this.isregister = true;
                          if (this.nom == "" || this.password == "" || this.prenoms == "" || this.email == "") {
                              Swal.fire({
                                  icon: 'error',
                                  title: 'Veuillez remplir correctement les champs',
                              });
                              this.error = true;
                              this.isregister = false;
                          } else {
                              let formData = new FormData();
                              formData.append('email', this.email);
                              formData.append('password', this.password);
                              formData.append('nom', this.nom);
                              formData.append('prenoms', this.prenoms);
                              formData.append('prof', this.prof);
                              axios.defaults.xsrfCookieName = 'csrftoken';
                              axios.defaults.xsrfHeaderName = 'X-CSRFToken';
                              axios.post(this.base_url + this.url_post,
                                  formData,
                                  {
                                      headers: {
                                          'Content-Type': 'multipart/form-data',
                                      }
                                  }).then((response) => {
                                  this.isregister = false;
                                  if (response.data.succes) {
                                      Swal.fire({
                                          icon: 'success',
                                          title: response.data.message,
                                          showConfirmButton: false,
                                          timer: 1500
                                      });

                                      if (response.data.is_prof) {
                                          window.location.replace(this.annonce)
                                      } else {
                                          window.location.replace(this.accueil)
                                      }    
                                  } else {
                                      Swal.fire({
                                          icon: 'error',
                                          title: response.data.message,
                                          showConfirmButton: false,
                                          timer: 1500
                                      });
                                  }
                                  console.log("success variable" + this.isSuccess)
                              })
                                  .catch((err) => {
                                      console.log(err);
                                      this.isregister = false;
                                  })
                          }
                      }
                  },

              }
          });
      </script>
# Post Vus js avec upload d'image et de fichier pdf
      <script>
          var app = new Vue({
              el: "#post_prof",
              data: {
                  isregister: false,
                  error : false,
                  isSuccess: false,

                  nom:"",
                  prenoms: "",
                  sexe:"",
                  date_naissance:"",
                  nationalite: "",
                  telephone: "",
                  email: "",
                  discipline: "",
                  cv: "",
                  init:"",
                  file:"",
                  input:"",
                  messages: "",
                  message:'',
                  pays: "",
                  base_url: window.location.protocol + "//" + window.location.host,
                  url_post: "{% url 'post_proffesseur' %}",
              },
              delimiters: ["${", "}"],
              mounted() { 
                  this.input = document.querySelector("#phone");
                  this.iti = intlTelInput(this.input, {
                      initialCountry: "ci"
                  });
                  console.log(this.iti.getNumber())
                  this.pays = $("#country_selectorbt").countrySelect({
                      preferredCountries: ['ci','fr','ml','ca','us', 'bf',],
                      responsiveDropdown:true 
                  });
                  this.nationalite = this.pays[0]['value']
              },
              methods: {
                  handlefile: function(){
                      this.file = this.$refs.file.files[0];
                  },

                  cvfile: function(){
                      this.cv = this.$refs.cvfile.files[0];
                  },
                  registerProf: function(){
                      this.telephone = this.iti.getNumber()
                      this.nationalite = this.pays[0]['value']
                      if (!this.isregister) {
                          this.isregister = true;
                          if (this.nom == "" || this.prenoms == "" || this.sexe == ""  || this.date_naissance == ""  || this.telephone == "" || this.email == "" || this.discipline == "" || this.messages == "") {
                              this.message = "Veuillez remplir tous les champs du formulaire s'il vous plaît."
                              this.isSuccess = false
                              this.error = true;
                              this.isregister = false;
                          } else {
                              let formData = new FormData();
                              formData.append('nom', this.nom);
                              formData.append('prenoms', this.prenoms);
                              formData.append('sexe', this.sexe);
                              formData.append('photo', this.file);
                              formData.append('date_naissance', this.date_naissance);
                              formData.append('nationalite', this.nationalite);
                              formData.append('telephone', this.telephone);
                              formData.append('email', this.email);
                              formData.append('discipline', this.discipline);
                              formData.append('cv', this.cv);
                              formData.append('message', this.messages);

                              axios.defaults.xsrfCookieName = 'csrftoken';
                              axios.defaults.xsrfHeaderName = 'X-CSRFToken';
                              axios.post(this.url_post,
                                  formData,
                                  {
                                      headers: {
                                          'Content-Type': 'multipart/form-data',
                                      }
                                  }).then((response) => {
                                  if (response.data.succes) {
                                      this.isSuccess = true
                                      this.isregister = false;
                                      this.error = false

                                      this.message = response.data.message
                                      this.nom = ""
                                      this.prenoms = ""
                                      this.masculin = ""
                                      this.feminin = ""
                                      this.photo = ""
                                      this.date_naissance = ""
                                      this.nationalite = ""
                                      this.telephone = ""
                                      this.email = ""
                                      this.discipline = ""
                                      this.cv = ""
                                      this.messages = ""

                                  } else {
                                      this.message = response.data.message
                                      this.error = true;
                                      this.isSuccess = false
                                      this.isregister = false;
                                  }
                                  console.log("success variable" + this.isSuccess)
                              })
                              .catch((err) => {
                                  console.log(err);
                                  this.isregister = false;
                              })
                          }
                      }
                  },

              }
          });
      </script>
# dans le html pour les inputs images et fichier pdf
      <div class="col-md-6">
        <label for="cv">Votre Curriculum Vitae</label>
        <input v-on:change="cvfile()" ref="cvfile" type="file" id="cv">
      </div>


       <label for="file-ip-1">
          Choisir une photo
      </label>
      <input ref="file" v-on:change="handlefile()" type="file" id="file-ip-1" accept="image/*" onchange="showPreview(event);">


# SCROLL INFINIE VU JS
     <script src='https://cdn.jsdelivr.net/npm/vue/dist/vue.js'></script>
      <script src='https://unpkg.com/axios/dist/axios.min.js'></script>

      <script>
          var app = new Vue({
              el: '#agenda_id',
              data: {
                  post_url:"{% url 'get_agenda' %}",
                  items: [],
                  endOfTheScreen: false,
                  isLoading: false,
                  pageno: 1,
                  limit: 1,
                  posts: [],
                  affiches: [],
                  showbtn: false,
                  startIndex:0,
                  endIndex : 3,
                  autoLoading: true,
                  loadder :false,
                  first_load: true,
              },
              delimiters: ["${", "}"],
              watch: {
                  endOfTheScreen(endOfTheScreen) {
                      if (endOfTheScreen === true && this.autoLoading) {

                          if(this.affiches.length < this.posts.length){
                              this.loadder = true;
                              setTimeout(()=>{
                                  this.loadder = false;
                                  this.afficheModule(this.startIndex, this.endIndex);
                              }, 2000)

                          }    
                      }
                  }
              },
              created() {
                  window.onscroll = () => {
                      this.endOfTheScreen = this.scrollCheck();
                  };
              },
              async mounted() {
                  await this.affichageAgenda();

              },
              updated: function () {
                  this.$nextTick(function () {
                    this.first_load = false
                  })
                },
              methods: {
                  scrollCheck() {
                      return window.scrollY + window.innerHeight === document.documentElement.offsetHeight;
                  },
                  affichageAgenda: function() {
                      this.showbtn = false;
                      this.isLoading = true;
                      axios.defaults.xsrfCookieName = 'csrftoken';
                      axios.defaults.xsrfHeaderName = 'X-CSRFToken';
                      axios.get(this.post_url).then(response => {
                          this.items = response.data.agendas
                          for (let i in this.items) {
                              console.log(i, "postrd")
                              if (i <= this.items.length){
                                  this.posts.push(this.items[i]);  
                              }

                          }
                          this.isLoading = false;
                          this.showbtn = true;
                          this.afficheModule(this.startIndex, this.endIndex)
                      }).catch((err) => {
                          console.log(err)
                      })
                  },
                  afficheModule: function(start , end) {
                      this.affiches = [...this.affiches , ...this.posts.slice(start,end)] 
                      console.log(this.affiches, this.endIndex)
                      if(this.affiches.length >0 && this.posts.length > end){
                              this.startIndex = end 
                              this.endIndex = end+3

                          }

                  },
                  truncate: function(text){
                      let length = 250
                      let suffix = '...'
                      if (text.length > length) {
                          return text.slice(0, length) + suffix;
                      } else {
                          return text;
                      }
                  }
              },
          })
      </script>

# SCROLL INFINIE GRAPHQL

      <script>
        var ma_vue2 = new Vue({
          el: '#graph_service',
          data: {
            resultats :[],
            base_url: window.location.protocol + "//" + window.location.host + '/',
            next: false,
            is_start: false,
            after: '',
          },
          delimiters: ["${", "}"],
          mounted () {
            this.getservices()
          },
          created(){
            window.addEventListener('scroll', (e)=>{
                this.verif()
            })
          },
          methods: {
            verif: function(e){
              if (this.is_start){return}

              var bod = document.querySelector("body")
              if (window.scrollY >= parseInt((bod.clientHeight * 50) / 100)){
                if (this.next){
                  this.get_next_services()
                }

              }
            },
            getservices: function () {
              this.is_start = true
              axios.defaults.xsrfCookieName = 'csrftoken'
              axios.defaults.xsrfHeaderName = 'X-CSRFToken'
              axios({
                url : this.base_url + 'graphql',
                method : 'post',
                data : {
                  query: `
                  query{
                    allServices(first:3 status:true){
                      edges{
                        node{
                          id
                          titre
                          icon
                          description
                          status
                        }
                        cursor
                      }
                      pageInfo{
                        endCursor
                        hasNextPage
                      }
                    }
                  }
                  `
                }
              }).then(response => {
                this.resultats = response.data.data.allServices.edges;  
                this.next = response.data.data.allServices.pageInfo.hasNextPage;  
                this.after = response.data.data.allServices.pageInfo.endCursor;  
                this.is_start = false;  
                console.log(response.data.data.allServices.edges); 
              })
              .catch((err) => {
                // console.log(err)
              })
            },

            get_next_services: function () {
              this.is_start = true
              axios.defaults.xsrfCookieName = 'csrftoken'
              axios.defaults.xsrfHeaderName = 'X-CSRFToken'
              axios({
                url : this.base_url + 'graphql',
                method : 'post',
                data : {
                  query: `
                    query{
                      allServices(first:3 after:"` + this.after + `" status:true){
                        edges{
                          node{
                            id
                            titre
                            icon
                            description
                            status
                          }
                          cursor
                        }
                        pageInfo{
                          endCursor
                          hasNextPage
                        }
                      }
                    }
                  `
                }
              }).then(response => {
                this.resultats = this.resultats.concat(response.data.data.allServices.edges);
                this.next = response.data.data.allServices.pageInfo.hasNextPage;  
                this.after = response.data.data.allServices.pageInfo.endCursor;  
                this.is_start = false;  
                console.log(response.data.data.allServices.edges); 
              })
              .catch((err) => {
                // console.log(err)
              })
            },
          }
        })

      </script>

# SCROLL INFINIE API REST OU PAGINATION API Rest
  https://medium.com/@fk26541598fk/django-rest-framework-apiview-implementation-pagination-mixin-c00c34da8ac2
