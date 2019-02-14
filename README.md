# GoogleVision_C-
Accessing Google Vision API using c#


public async Task<IHttpActionResult> performGoogleVision(string imageurl)
        {
            Uri URL = new Uri("https://vision.googleapis.com/v1/images:annotate?key={your google vision key}");
            
            // You need to build your own Model Classes based on Google Vision Response
            var googleVisionObject = new GoogleVision.GoogleVisionObject();
            var googleVisionObjectSimplified = new GoogleVision.GoogleVisionSimplified();
            var json = "";            

            HttpResponseMessage response = null;
            WebRequestHandler handler = new WebRequestHandler();

            List<Feature> GVRFeatures = new List<Feature>();
            GVRFeatures.Add(new Feature { type = "LABEL_DETECTION", maxResults = 3 });
            GVRFeatures.Add(new Feature { type = "FACE_DETECTION", maxResults = 2 });            
            GVRFeatures.Add(new Feature { type = "TEXT_DETECTION", maxResults = 2 });
            GVRFeatures.Add(new Feature { type = "LOGO_DETECTION", maxResults = 2 });            
                          
            string ImageString = ImageToBase64("aaa", imageurl);

            var GVRRequest = new GoogleVisionRequest();
            GVRRequest.requests = new List<Request>
            {
             new Request { image = new GoogleVision.Models.Image { content = ImageString }, features = GVRFeatures  }
            };            

            using (var client = new HttpClient(handler))
            {                

                client.BaseAddress = URL;
                client.DefaultRequestHeaders.Accept.Clear();
                client.DefaultRequestHeaders.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));

                try
                {
                    response = await client.PostAsJsonAsync("", GVRRequest);                    
                    if (response.IsSuccessStatusCode)
                    {                      
                        
                        json = await response.Content.ReadAsStringAsync(); //get Json
                        googleVisionObject = Newtonsoft.Json.JsonConvert.DeserializeObject<Models.GoogleVision.GoogleVisionObject>(json);

                        
                        if (googleVisionObject != null)
                        {                        
                        googleVisionObjectSimplified.fulltext = googleVisionObject.responses[0].textAnnotations[0].description;
                        googleVisionObjectSimplified.textannotations = googleVisionObject.responses[0].textAnnotations.ToString();    
                                             
                        googleVisionObjectSimplified.alldata = json;
                        googleVisionObjectSimplified.webdetection = googleVisionObject.responses[0].webDetection.ToString();
                        googleVisionObjectSimplified.faceannotations = googleVisionObject.responses[0].faceAnnotations.ToString();

                        }   
                    }
                    
                    else
                    {
                       
                    }
                }
                catch (Exception ex)
                {
                    response.Content = new StringContent(ex.Message);                    
                    response.Content = new StringContent("Error");                   
                    
                }               
               
                return Json (googleVisionObjectSimplified);
                }
        }
