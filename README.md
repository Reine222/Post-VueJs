# Post-VueJs
Vue JS

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
