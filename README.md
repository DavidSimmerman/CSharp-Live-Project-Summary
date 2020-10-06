# CSharp-Live-Project-Summary

## Introducton
While I was enrolled at The Tech Academy, I was given the opportunity to participate in a two-week live project sprint. The project uses C# and the ASP.net framework. I, as well as several other students, was tasked with completing stories that contributed to a website. We could choose from several assignments that pertained to both the front and back ends, which included UX/UI, adding new features, and squashing bugs that currently existed. These stories were leveled from 2, 3, 5, and 8 and contributed to a real website for a theater and acting company. All of the students included in the sprint were treated as if they were employees and had to attend several mandetory meetings. Each of our stories had to be reviewed before they were merged into the master branch and were often denied with a list of changes that needed to be made. The project was designed to simulate a real software developer work environment to prepare students for their first job.

Below, I have detailed each of the stories I had worked on during these two weeks and my experience/takeaways.

## Default Photo Deletion
The first story I worked on was simple and was intended to familiarize new students to the program. All that was asked was to figure out where the default photo was set to null and to remove or comment out that line of code since the null reference caused an error. I quickly found how massive the project was and how difficult it was to find the section of code I needed to change. I started going through and understanding how the program functioned. I eventually found the line that was causing the problem. Fortunately, the fix was easy: I simply had to comment out one line. Although this fixed the crashing error, it presented a few visual errors. I took this up with my instructor who created a new story for me that was marked at level 3. Now, when a default photo was deleted, I had to make the program set the next available photo. If there were no remaining photos, I had to set it to a 'Photo Unavailable' image. I was able to swiftly write the following code with very little errors or troubles.
<details>
  <summary>Default Photo Reasign on Deletion</summary>

  ```
  // Checks for a production using this photo as a deleted photo
  Production production = db.Productions.FirstOrDefault(x => x.DefaultPhoto.PhotoId == photo.PhotoId);
  if (production != null)  // If Production exists
  {   
      // Checks to see if there is another photo related to the production
      if (production.ProductionPhotos.Where(p => p.PhotoId != null).Count() > 0)
      {
          foreach (var potentialDefaultPhoto in production.ProductionPhotos)
          {
              if (potentialDefaultPhoto.PhotoId == photo.PhotoId) continue;  // Ignores current default photo
              // Error handling for deleted photos that still have references
              else if (potentialDefaultPhoto == null || potentialDefaultPhoto.PhotoId == null) continue;
              else
              {
                  // Sets new default photo
                  production.DefaultPhoto = potentialDefaultPhoto;
                  break;  // Exists the loop since a photo has been found
              }
          }
      }
      // Sets the default photo to "Photo Unavailable"
      else production.DefaultPhoto = db.ProductionPhotos.Where(p => p.Title == "Photo Unavailable").FirstOrDefault();
      DbEntityEntry<Production> dbEntityEntry = db.Entry(production);
      dbEntityEntry.CurrentValues.SetValues(production);
  }
  ```

</details>

## Recent Definition
The next story I took was slightly more complicated since it was a level 5 story. I was required to add a field to the admin settings that would allow them to set a time period for the most recent donors to the website. The biggest challenge I faced was simply figuring out how the form was created and settings were stored. Fortunately, this didn't prove to be too dificult since I was able to reference the surrounding code to create my own. I created two radio inputs, one for a date time and the other for a span, and then created the inputs for each. This way the admin could pick which type they wanted to use while also setting the span or date. I then created a function in the AdminController that read the information and saved it in the AdminSettings json file.
<details>
  <summary>Recent Definition HTML/Razor</summary>
  
  ```
  RECENT DEFINITION:
  <br />
  <input type="radio" name="recent_definition.bUsingSpan" value="false" id="recentDefDateSel" 
         @if (!Model.recent_definition.bUsingSpan) { 
             @:checked 
             }/>

  <label for="recentDefDateSel">Date:</label>
  <br />
  <input align="center" type="date" name="recent_definition.date" value="@Model.recent_definition.date.ToString("yyyy-MM-dd")">
  
  <br />
  <br />

  <input type="radio" name="recent_definition.bUsingSpan" value="true" id="recentDefSpanSel" 
             @if (Model.recent_definition.bUsingSpan) { 
                 @:checked 
                 }/>

  <label for="recentDefDateSel">Span (Months):</label>

  <br />

  <input align="center" type="number" name="recent_definition.span" value="@Html.DisplayFor(model => model.recent_definition.span)">
  ```
  
</details>
<details>
  <summary>Recent Definition C#</summary>
  
  ```
  //Updates Changes in database depending on inputs from the Admin Settings form for recent_definition.
  private void UpdateSubscribers()
  {
      // Retrieve Admin Settings
      dynamic adminSettings = AdminSettingsReader.CurrentSettings();

      // Init variable
      DateTime recentDef = DateTime.Now;

      // Check and see what the recent definition is
      // Then set recentDef to the correct DateTime
      // If there is no selection, recentDef will be set to now
      // No recent subs will be displayed unless they donated at the exact time
      if (!adminSettings.recent_definition.bUsingSpan)
      {
          recentDef = adminSettings.recent_definition.date;
      }
      else if (adminSettings.recent_definition.bUsingSpan)
      {
          recentDef = recentDef.AddMonths(-Convert.ToInt32(adminSettings.recent_definition.span));
      }

      foreach (var subscriber in db.Subscribers)
      {
          if (recentDef >= subscriber.LastDonated)
          {
              subscriber.RecentDonor = false;
          }
          else
          {
              subscriber.RecentDonor = true;
          }
      }
      db.SaveChanges();
  }
  ```
  
</details>

## Favorite Cast Members Button
For my final story, I decided to challenge myself with a level 8 task. I had to create a button for the user to favorite a cast member and save that in the database. The front end was not very difficult at all. I used font-awesome to create a heart icon that could be an outline or filled in depending on the cast member's status. I then created a string property for all users that would store the initials of each favorited cast member. After that, I made a function that would add or remove a cast member's initials from the string if they were favorited.
<details>
  <summary>Toggle Cast Members C#</summary>
  
  ```
  public void ToggleFavoriteCastMembers(int? id)
  {
      string cmId = id.ToString();

      // Check/get the current user
      if (Request.IsAuthenticated)
      {
          var userManager = new UserManager<ApplicationUser>(new UserStore<ApplicationUser>(db));
          ApplicationUser currentUser = userManager.FindById(User.Identity.GetUserId());

          if (currentUser != null && currentUser.FavoriteCastMembers != null)
          {
              // Break down the FavoriteCastMembers string and turn it into a list
              List<string> favCastIds = new List<string> { };
              if (currentUser.FavoriteCastMembers != "")
                  favCastIds = currentUser.FavoriteCastMembers.Split(',').ToList();
              // If the Id is in the list, remove it. If not, add it.
              if (favCastIds.Contains(cmId)) favCastIds.Remove(cmId);
              else favCastIds.Add(cmId);

              // Convert the list back into a string to store in the database
              currentUser.FavoriteCastMembers = string.Join(",", favCastIds);
              db.SaveChanges();
          }
          else if (currentUser != null)
          {
              currentUser.FavoriteCastMembers = cmId;
              System.Diagnostics.Debug.WriteLine("BBGG");
              db.SaveChanges();
          }
      }
  }
  ```

</details>
This wasn't a huge struggle to make, but I got a little stuck trying to figure out how to access the current user. I eventually found another part of the site that accessed the user's object which was the solution. Once that was completed and worked well, I was done with the story; however, there was one thing I wanted to change. As it stood currently, the page would refresh every time the user clicked the favorite button. I wasn't a fan of that and wanted it to be seamless, so I added some AJAX to remove the refresh.
<details>
  <summary>Javascript and AJAX</summary>

```
function FavoriteCastMember(castMemeberId) {
    var paragraphId = "fb-" + castMemeberId;
    var likeBtn = document.getElementById(paragraphId);

    $.ajax({
        method: 'POST',
        url: '/Account/ToggleFavoriteCastMembers/',
        data: { 'id': castMemeberId },

    })

    if (likeBtn.classList.contains("fb-not-favorited")) {
        likeBtn.classList.remove("fb-not-favorited");
        likeBtn.classList.add("fb-favorited");
    } else if (likeBtn.classList.contains("fb-favorited")) {
        likeBtn.classList.remove("fb-favorited");
        likeBtn.classList.add("fb-not-favorited");
    }
};
```

</details>

## Conclusion
This two week live project was very beneficial for me. I learned a lot about the .NET framework and AJAX and it gave me a great idea of how a real job will function. My biggest struggle during the project was not knowing how the program worked together. It was a massive website with tons of files, so sometimes I was lost on how to access certain info or where a certain function was located. In a real job scenario, I would have more than two weeks to familiarize myself with the project, so I don't see this being a problem in the future. I thoroughly enjoyed challenging and pushing myself through these stories to learn more and I certainly picked up a lot of new tips and tricks with C# and software development in general. I was also able to meet and connect with other students and software developers and hear about their experiences in Computer Science.
