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
