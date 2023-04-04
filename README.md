# CvJava



#Ear

C'est le projet à deployer il utilise les deux autres commme dependance.

#EJB

On a dans EJBmodule 4 packages:

-Le package pour entities: on a Compte pour l'authentification, Cv ,ExperiencePro 
Ils sont annotés avec @Entity 


-Le package pour DAO: Les interfaces et leurs implementations pour chaque entité et on a bien sûr une interface générique et son implementation qui regroupe les methodes
que les autres interfaces ont en commun.
Les interfaces sont annotées avec @Local et les implementations de ces interfaces avec @Stateless


-Le package pour Mail sa configuration est comme tel:

	public boolean sendEmail(String to, String subject, String text) {
		  	boolean flag = false;
		  	final String username = "traoreharouna04@gmail.com";
	        final String password = "gaxssstmtnhgqdod";
	        
	        Properties properties = System.getProperties();
	        properties.put("mail.smtp.host", "smtp.gmail.com");
	        properties.put("mail.smtp.port", "465");
	        properties.put("mail.smtp.ssl.enable", "true");
	        properties.put("mail.smtp.auth", "true");
	        

	       
	        
	        
	        //session
	        Session session = Session.getInstance(properties, new javax.mail.Authenticator() {
	            @Override
	            protected PasswordAuthentication getPasswordAuthentication() {
	                return new PasswordAuthentication(username, password);
	            }
	        });
	        
	     // Used to debug SMTP issues
	        session.setDebug(true);
	        try {

	        	MimeMessage message = new MimeMessage(session);
	            message.setRecipient(Message.RecipientType.TO, new InternetAddress(to));
	            message.setFrom(new InternetAddress(username));
	            message.setSubject(subject);
	            message.setText(text);
	            Transport.send(message);
	            flag = true;
	        } catch (Exception e) {
	            e.printStackTrace();
	        }


		 
		return flag;
	}


- Le package pour pdf(non finit car mal configuration empeche le serveur de demarrer correctement):


public class PdfImpl {
	    private static String FILE = "C:\\test.pdf";
	    private static Font catFont = new Font(Font.FontFamily.TIMES_ROMAN, 18,
	            Font.BOLD);
	    private static Font redFont = new Font(Font.FontFamily.TIMES_ROMAN, 12,
	            Font.NORMAL, BaseColor.RED);
	    private static Font subFont = new Font(Font.FontFamily.TIMES_ROMAN, 16,
	            Font.BOLD);
	    private static Font smallBold = new Font(Font.FontFamily.TIMES_ROMAN, 12,
	            Font.BOLD);

	    private static void addMetaData(Document document) {
	        document.addTitle("My first PDF");
	        document.addSubject("Using iText");
	        document.addKeywords("Java, PDF, iText");
	        document.addAuthor("Lars Vogel");
	        document.addCreator("Lars Vogel");
	    }
	    public  void imprimer() {
	        try {
	            Document document = new Document();
	            PdfWriter.getInstance(document, new FileOutputStream(FILE));
	            document.open();
	            addMetaData(document);
	            
	            document.close();
	        } catch (Exception e) {
	            e.printStackTrace();
	        }
	        
	    }
}


#Web

-Nous avons les controllers dans le package com.groupeisi.controllers:
-loginServlet: gère l'authentification, la création de la session et dirige vers la page d'accueil.
-logoutServlet: récupère la session, la détruit et renvoie dans la page de connexion
-CvServlet: gère la l'ajout, suppression, modification des infos et l'affichage (de tous les cv ou un cv).
il utilise l'interface ICv dans ejb comme tel:
   @EJB
	private ICv icv;
	Cv cv = new Cv();
    
    recuperation d'un seul cv :
                if(idP!=null) {
	    		     Cv cv0 = icv.gets(Integer.parseInt(idP), cv);
	    		 
	    			  request.setAttribute("cvs", cv0);
	    			  this.getServletContext().getRequestDispatcher("/cv/new.jsp").forward(request, response); 
	    		 
	    	  }
                 
   recuperation de tous les cv  :
       
            int id = (int) session.getAttribute("id");//Comme on recupere les cvs de la personne connectée
		  			List<Cv> cvs = icv.cvs(id);
		  			 request.setAttribute("cvs", cvs);
		  			 this.getServletContext().getRequestDispatcher("/cv/cv.jsp").forward(request, response);

                
 -Suppression:
             if(idD!=null) {
	    		  int result = icv.delete(Integer.parseInt(idD), cv);
	    		  if(result==1) {
	    			  request.setAttribute("success", "Cv supprimé avec succès");
	    			  response.sendRedirect("cvs");  
	    		  }
	    	  }
 -Modification des infos:
       if(cvid!=null && id != null && nom !=null && prenom != null && age != null && telephone!= null && adresse != null && specialite != null && niveauEtude != null  ) {
			 Compte cp = icompte.gets(Integer.parseInt(id), compte);
			 cv.setAdresse(adresse);
			 cv.setAge(Integer.parseInt(age));
			 cv.setNiveauEtude(niveauEtude);
			 cv.setNom(nom);
			 cv.setPrenom(prenom);
			 cv.setSpecialite(specialite);
			 cv.setTelephone(telephone);
			 cv.setId(Integer.parseInt(cvid));
			 cv.setCompte(cp);
			 int result=icv.update(cv);
			 if(result==1) {
				 request.setAttribute("success", "Modification réussi");
				 response.sendRedirect("cvs");
				
				
			 }
			 
		 }
  -Ajout:
  
  	if(cvid==null && id != null && nom !=null && prenom != null && age != null && telephone!= null && adresse != null && specialite != null && niveauEtude != null  ) {
			 cv.setAdresse(adresse);
			 cv.setAge(Integer.parseInt(age));
			 cv.setNiveauEtude(niveauEtude);
			 cv.setNom(nom);
			 cv.setPrenom(prenom);
			 cv.setSpecialite(specialite);
			 cv.setTelephone(telephone);
			 Compte cp = icompte.gets(Integer.parseInt(id), compte);
			 cv.setCompte(cp);
			 int result=icv.add(cv);
			 if(result==1) {
				 request.setAttribute("success", "Enregistrement réussi");
				 response.sendRedirect("cvs");
				
				
			 }
			 
		 }
		
 
 
   Tous les autres ont le même comportement.
   
   
   Et enfin on a les vues dans webapp:
   
   Exemple d'une vue d'affichage:    
            <c:forEach var="cvs" items="${cvs}">
                <tr >
                  
                    <tr >
                    
               
                    <td><c:out value="${cvs.nom}" /></td>
                    <td><c:out value="${cvs.prenom}" /></td>
                    <td><c:out value="${cvs.compte.email}" /></td>  
                     <td><c:out value="${cvs.telephone}" /></td>
                       <td><c:out value="${cvs.age}" /></td>
                     
                   <td>
                          <a type="button" href="cvs?idD=<c:out value="${cvs.id}" />" class="btn btn-outline-danger btn-sm px-2">
                       Delete
                          
                          </a>
                         
                           <a type="button" href="cvs?idP=<c:out value="${cvs.id}" />" class="btn btn-outline-info btn-sm px-2">
                           Plus ...
                          </a>
                          
                           <a type="button" href="cvs?idI=<c:out value="${cvs.id}" />" class="btn btn-outline-info btn-sm px-2">
                           Imprimer
                          </a>
                          
                          <a type="button" href="exp?cvid=<c:out value="${cvs.id}" />" class="btn btn-outline-primary btn-sm px-2">
                           Experiences
                          </a>
                          
                          
                        </td>
                   

                </tr>
                   

              
                 </c:forEach>
                
             
             
Exemple d'un formulaire:

       <% if(request.getAttribute("cvs") == null){%>
            <form action="cvs" method="post" >
              <div class="row">
                <div class="col-md-6 form-group">
                       <input hidden class="fadeIn second" name="id" value="<%= session.getAttribute("id") %>" >
                
                  <input type="text" name="nom" class="form-control" id="nom" placeholder="NOM" required>
                </div>
                <div class="col-md-6 form-group mt-3 mt-md-0">
                  <input type="text" class="form-control" name="prenom" id="prenom" placeholder="PRENOM" required>
                </div>
              </div>
              <div class="row">
                <div class="col-md-6 form-group">
                  <input type="number" name="age" class="form-control" id="age" placeholder="AGE" required>
                </div>
                <div class="col-md-6 form-group mt-3 mt-md-0">
                  <input type="text" class="form-control" name="adresse" id="adresse" placeholder="ADRESSE" required>
                </div>
              </div>
              
              <div class="row">
                <div class="col-md-6 form-group">
                  <input type="text" name="specialite" class="form-control" id="specialite" placeholder="SPECIALITE" required>
                </div>
                <div class="col-md-6 form-group mt-3 mt-md-0">
                    <select class="form-select " id="niveauEtude" name="niveauEtude" >
					  <option selected >Niveau d'étude</option>
					  <option value="Bac">Bac</option>
					  <option value="Bac+1">Bac+1</option>
					  <option value="Bac+2">Bac+2</option>
					   <option value="Bac+3">Bac+3</option>
					  <option value="Bac+4">Bac+4</option>
					  <option value="Bac+5">Bac+5</option>
					</select>
                </div>
              </div>
              
             
              <div class="text-center"><button class="btn btn-primary" type="submit">Enregistrer</button></div>
            </form>
            <%}%> 
         <% if(request.getAttribute("cvs") != null){%>

 
 
         
            <form action="cvs" method="post" >
              <div class="row">
                <div class="col-md-6 form-group">
                 <input type="text" hidden name="cvid" class="form-control" id="cvid"  value="${cvs.id}" required>
                  <input type="text" name="nom" class="form-control" id="nom" value="${cvs.nom}" required>
                </div>
                <div class="col-md-6 form-group mt-3 mt-md-0">
                  <input type="text" class="form-control" name="prenom" id="prenom" value="${cvs.prenom}" required>
                </div>
              </div>
              <div class="row">
                <div class="col-md-6 form-group">
                  <input type="number" name="age" class="form-control" id="age" value="${cvs.age}" required>
                </div>
                <div class="col-md-6 form-group mt-3 mt-md-0">
                  <input type="text" class="form-control" name="adresse" id="adresse" value="${cvs.adresse}" required>
                </div>
              </div>
              
              <div class="row">
                <div class="col-md-6 form-group">
                  <input type="text" name="specialite" class="form-control" id="specialite" value="${cvs.specialite}" required>
                </div>
                <div class="col-md-6 form-group mt-3 mt-md-0">
                    <select class="form-select " id="niveauEtude" name="niveauEtude" >
					  <option selected >${cvs.niveauEtude}</option>
					  <option value="Bac">Bac</option>
					  <option value="Bac+1">Bac+1</option>
					  <option value="Bac+2">Bac+2</option>
					   <option value="Bac+3">Bac+3</option>
					  <option value="Bac+4">Bac+4</option>
					  <option value="Bac+5">Bac+5</option>
					</select>
                </div>
              </div>
              
              
              <div class="text-center"><button class="btn btn-primary" type="submit">Modifier</button></div>
            </form>
            <%}%> 
              
                  
               Merci d'avoir prie le temps de regarder tout se bla bla!!!
   
   
    
    

