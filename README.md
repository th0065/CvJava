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
             <c:forEach var="stud" items="${listStudent}">
                <tr >
                 <th scope="row">
                          <div class="form-check">
                            <input class="form-check-input" type="checkbox" value="" id="qq" />
                          </div>
                 </th>
                    <td><c:out value="${stud.firstName}" /></td>
                    <td><c:out value="${stud.lastName}" /></td>
                    <td><c:out value="${stud.birthdate}" /></td>
                     <td><c:out value="${stud.phone}" /></td>
                   
                   <td>
                          <a type="button" href="student?idD=<c:out value="${stud.id}" />" class="btn btn-danger btn-sm px-3">
                       Delete
                          
                          </a>
                         
                           <a type="button" href="student?idM=<c:out value="${stud.id}" />" class="btn btn-info btn-sm px-3">
                           Plus ...
                          </a>
                          
                           <a type="button" href="student?idP=<c:out value="${stud.id}" />" class="btn btn-primary btn-sm px-3">
                            Update
                          </a>
                        </td>
                   

                </tr>
            </c:forEach>
                  
                    </tbody>
                  </table>
           <nav aria-label="Navigation for students">
        <ul class="pagination">
            <c:if test="${page != 1}">
                <li class="page-item"><a class="page-link"
                    href="student?page=${ page - 1 }">Previous</a>
                </li>
            </c:if>

            <c:forEach begin="1" end="${pages}" var="i">
                <c:choose>
                    <c:when test="${ page == i}">
                        <li class="page-item active"><a class="page-link">
                                ${i} <span class="sr-only">(current)</span></a>
                        </li>
                    </c:when>
                    <c:otherwise>
                        <li class="page-item"><a class="page-link"
                            href="student?page=${i}">${i}</a>
                        </li>
                    </c:otherwise>
                </c:choose>
            </c:forEach>

            <c:if test="${page lt pages}">
                <li class="page-item"><a class="page-link"
                    href="student?page=${page+1}">Next</a>
                </li>
            </c:if>
        </ul>
    </nav>
             
             
             
Exemple d'un formulaire:

     <form method="post" action="student" class="m-5">
                 
                   <% if(request.getAttribute("success") != null){%>
                <div class="row form-group">
                     <hr>
                    <div class="col-md-6 ">
                        <span class='text-success '><%= request.getAttribute("success") %></span>
                    </div>
                     <hr>
                </div>
                    <%}%> 
                     <% if(request.getAttribute("student") == null){%>
                
                   
                  <table class="table table-borderless mb-0">
                    <thead>
                      <tr>
                        
                        <th scope="col"> 
                        <div class="form-group">
                          <label>FirstName :</label>
                            <input class="form-input" type="text"  id="firstName" name="firstName" />
                          </div>
                          </th>
                         
                        <th scope="col">
                         <div class="form-group">
                          <label>LastName  :</label>
                            <input class="form-input" type="text"  id="lastName" name="lastName" />
                          </div>
                          </th>
                           </tr>
                          <tr>
                        <th scope="col">
                         <div class="form-group">
                          <label>BirthDate :</label>
                            <input class="form-input" type="date"   id="date"  name="date" />
                          </div>
                          </th>
                          
                          <th scope="col">
                         <div class="form-group">
                          <label>Phone Number :</label>
                            <input class="form-input" type="text"  id="phone" name="phone" />
                          </div>
                          </th>
                       
                      </tr>
                      
                     
                        
                        
                    </thead>
                    <tbody>
                     
                      
            
                      
                    </tbody>
                  </table>
                  <div class="m-5 "> <button type="submit" class="btn btn-sm btn-success px-5">Enregistrer</button></div>
                   <%}%> 
                   
                    <% if(request.getAttribute("success") != null){%>
                <div class="row form-group">
                     <hr>
                    <div class="col-md-6 ">
                        <span class='text-success '><%= request.getAttribute("success") %></span>
                    </div>
                     <hr>
                </div>
                    <%}%> 
                     <% if((request.getAttribute("student") != null) && (request.getAttribute("detail") == null ) ){%>
                
                   
                  <table class="table table-borderless mb-0">
                    <thead>
                      <tr>
                        
                        <th scope="col"> 
                          <input class="" type="text" hidden value="<c:out value="${student.id}" />"  id="id" name="id" />
                        
                        <div class="form-group">
                          <label>FirstName :</label>
                            <input class="form-input" type="text" value="<c:out value="${student.firstName}" />"  id="firstName" name="firstName" />
                          </div>
                          </th>
                         
                        <th scope="col">
                         <div class="form-group">
                          <label>LastName  :</label>
                            <input class="form-input" type="text" value="<c:out value="${student.lastName}" />" id="lastName" name="lastName" />
                          </div>
                          </th>
                           </tr>
                          <tr>
                        <th scope="col">
                         <div class="form-group">
                          <label>BirthDate : </label>
                           
                          </div>
                           <input class="form-input" type="date" value="<c:out value="${student.birthdate}" />"   id="date"  name="date" />
                          </th>
                          
                          <th scope="col">
                         <div class="form-group">
                          <label>Phone Number :</label>
                            <input class="form-input" type="text" value="<c:out value="${student.phone}" />"  id="phone" name="phone" />
                          </div>
                          </th>
                       
                      </tr>
                      
                     
                        
                        
                    </thead>
                    <tbody>
                     
                      
            
                      
                    </tbody>
                  </table>
                  <div class="m-5 "> <button type="submit" class="btn btn-sm btn-success px-5">Enregistrer</button></div>
                   <%}%> 
                   
                    <% if( (request.getAttribute("detail") != null) && (request.getAttribute("student") != null ) ){%>
                
                   
                  <table class="table table-borderless mb-0">
                    <thead>
                      <tr>
                        
                        <th scope="col"> 
                          <input class="" type="text" hidden value="<c:out value="${student.id}" />"  id="id" name="id" />
                        
                        <div class="form-group">
                          <label>FirstName :</label>
                            <input class="form-input" type="text" value="<c:out value="${student.firstName}" />"  id="firstName" name="firstName" />
                          </div>
                          </th>
                         
                        <th scope="col">
                         <div class="form-group">
                          <label>LastName  :</label>
                            <input class="form-input" type="text" value="<c:out value="${student.lastName}" />" id="lastName" name="lastName" />
                          </div>
                          </th>
                           </tr>
                          <tr>
                        <th scope="col">
                         <div class="form-group">
                          <label>BirthDate : </label>
                         <input class="form-input" type="date" value="<c:out value="${student.birthdate}" />"  id="date" name="date" />
                           
                          </div>
                          
                          </th>
                          
                          <th scope="col">
                         <div class="form-group">
                          <label>Téléphone :</label>
                            <input class="form-input" type="text" value="<c:out value="${student.phone}" />"  id="phone" name="phone" />
                          </div>
                          </th>
                       
                      </tr>
                      
                      <tr> 
                        <th> <label>Insriptions</label></th>
                         </tr>
                         <tr>
                          <c:forEach var="inscr" items="${student.inscriptions}">
                         
                           <th scope="col">
                         <div class="form-group">
                          <label>Details :</label>
                            <input class="form-input" type="text" value="<c:out value="${inscr.details}" />"  id="phone" name="phone" />
                             </div>
                          </th>
                          
                          <th scope="col">
                         <div class="form-group">
                          <label>Année: </label>
                            <input class="form-input" type="text" value="<c:out value="${inscr.year.name}" />"  id="phone" name="phone" />
                         </div>
                         
                          </th>
                          </tr>
                         </c:forEach>
                         
                     
                      <tr>
                      <th> 
                      <a type="button" href="inscription?param=add" class="btn btn-sm btn-success px-5">Inscrire</a>
                      <br></th>
                       <th scope="col">
                        
                       <a type="button" href="student" class="btn btn-sm btn-danger px-5">Retour</a>
                         </th>
                        </tr>
                    </thead>
                    <tbody>
                     
                      
            
                      
                    </tbody>
                  </table>
                 
                   <%}%> 
                   
                   
                   
                  </form>
                  
               Merci d'avoir prie le temps de regarder tout se bla bla!!!
   
   
    
    

