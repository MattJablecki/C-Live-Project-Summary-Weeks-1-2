# C-Live-Project-Summary-Weeks-1-2

## Introduction
During my time at The Tech Academy, I was assigned to a two-week sprint working on software to be used as a way to "manage a collection of construction jobs. Admins will be able to create and distribute a weekly schedule assigning users to certain jobs. Users will be able to keep track of which job they are assigned to for the week. There will also be the ability for admins to post company news and announcements, and chat functionality for the users to socialize."
  
My assignments on the project all revolved around a "User List" of all users who have signed up to use the software. This encompasses everything from populating the user list, creating delete user functionality, allowing for "dropdowns" within the list that would allow an admin to change the user's role in the company and assign them to different work roles, and creating error pop-ups where necessary. I learned quite a lot during this experience, and by focusing solely on one aspect of the project, I really began to take ownership of the user list and wanted to see it through to completion.

Listed below are the stories I worked on, a brief description of their expectations, and the code I created to complete them.


## User Stories
* [Create User List View](#create-user-list-view)
* [Change User Role From List](#change-user-role-from-list)
* [Error Catching For User Delete](#error-catching-for-user-delete)
* [Set Worker Type Field](#set-worker-type-field)



### Create User List View
When I joined the project, there were existing "dummy users" to test functionality within the software. The users were stored in the database, so by simply setting them to an enumerable (see below), I was able to pass the list of users to a partial view. 

        public ActionResult _UserListPartial()
        {
            return PartialView(db.Users.AsEnumerable());
        }

A partial view was used because this list was to be populated in the admin dashboard, not its own dedicated page. By setting the model type with Razor at the top of the page, I was able to use Html helpers to add "DisplayNameFor" and "DisplayFor" for all the elements of the users in the database. A foreach loop captured all the users and the partial view was populated.


    @model IEnumerable<ConstructionNew.Models.ApplicationUser>
    <table class="table">
        <tr>
            <th>
                @Html.DisplayNameFor(model => model.UserName)
            </th>
            <th>
                @Html.DisplayNameFor(model => model.FName)
            </th>
            <th>
                @Html.DisplayNameFor(model => model.LName)
            </th>
            <th>
                @Html.DisplayNameFor(model => model.WorkType)
            </th>
        @foreach (var item in Model)
        {
            <tr>
                <td>
                    @Html.DisplayFor(modelItem => item.UserName)
                </td>
                <td>
                    @Html.DisplayFor(modelItem => item.FName)
                </td>
                <td>
                    @Html.DisplayFor(model => item.LName)
                </td>
                <td>
                    @using (Html.BeginForm("EditWork", "Account", FormMethod.Post, new { enctype = "multipart/form-data" }))
                    {
                        @Html.Hidden("Id", item.Id)
                        @Html.DropDownList("workType", new SelectList(Enum.GetValues(typeof(ConstructionNew.Enums.WorkType))),                                  item.WorkType.ToString())
                        <input type="submit" value="Submit" onclick="" />
                    }
                </td>
                <td>
                    @using (Html.BeginForm("EditRole", "Account", FormMethod.Post, new { enctype = "multipart/form-data" }))
                    {
                        @Html.Hidden("Id", item.Id)
                        @Html.DropDownList("UserRole", new List<SelectListItem>
                        {
                            new SelectListItem { Text = "Admin", Value = "Admin"},
                            new SelectListItem { Text = "Manager", Value = "Manager"},
                            new SelectListItem { Text = "Employee", Value = "Employee"}
                        }, item.UserRole)
                        <input type="submit" value="Submit" onclick="" />}
                </td>
                <td>@Html.ActionLink("Delete User", "Delete", new { id = item.Id })</td>
            </tr>
        }

    </table>


The last part of this task was to enable a delete function for the admin which would allow a user to be deleted and removed from the database directly from the user list. The delete function was called with an action link:

      @Html.ActionLink("Delete User", "Delete", new { id = item.Id })</td>

and the delete method itself was written as such:

        public ActionResult Delete(string id)
        {
            try
            {
                var del = (from ApplicationUser in db.Users where ApplicationUser.Id == id select ApplicationUser).FirstOrDefault();
                db.Users.Remove(del);
                db.SaveChanges();
            }
            catch (Exception ex)
            {
                TempData["shortMessage"] = "This user is on the schedule and therefore cannot be deleted.";
            }
            return RedirectToAction("Index", "Home");
        }


 
 ### Change User Role From List
Once the list was properly populated, the next task was to allow for an admin to update user roles directly from the user list. Using a dropdown, I was able to add all the possible user roles to the user list partial view. The Html.BeginForm helper allowed for a streamlined way to call the correct method and post the selected value to both the view and the database. 

     @using (Html.BeginForm("EditRole", "Account", FormMethod.Post, new { enctype = "multipart/form-data" }))
                {
                    @Html.Hidden("Id", item.Id)
                    @Html.DropDownList("UserRole", new List<SelectListItem>
                    {
                        new SelectListItem { Text = "Admin", Value = "Admin"},
                        new SelectListItem { Text = "Manager", Value = "Manager"},
                        new SelectListItem { Text = "Employee", Value = "Employee"}
                    }, item.UserRole)
                    <input type="submit" value="Submit">}

Then, the user would be passed to the "EditRole" method, which would change their user role in both the view and the database.
        
        public ActionResult EditRole(string id, string UserRole)
        {
            var user = (from ApplicationUser in db.Users where ApplicationUser.Id == id select ApplicationUser).FirstOrDefault();
            var current = user.UserRole;
            user.UserRole = UserRole;

            UserManager.RemoveFromRole(user.Id, current);
            UserManager.AddToRole(user.Id, UserRole);
            if (ModelState.IsValid)
            {
                db.Entry(user).State = EntityState.Modified;
                db.SaveChanges();
                return RedirectToAction("Index", "Home");
            }
            return RedirectToAction("Index", "Home");
        }
        

### Error Catching For User Delete
Something that was unforeseen was that deleting a user who was already on the schedule to work was not possible. This was ultimately a good thing, but there was no way to communicate that to the user. So by inserting a try/catch block, I created a message that would pop up when an admin tried to delete a scheduled user.  

        catch (Exception ex)
        {
            TempData["shortMessage"] = "This user is on the schedule and therefore cannot be deleted.";
        }


Then I passed the TempData to the view using JavaScript.

        var t = TempData["shortMessage"];
        
        @if (t != null)
        {
        <script type="text/javascript">
        window.onload = function () {
            alert("@ViewData["shortMessage"] This user is on the schedule, therefore cannot be deleted.")
        };
        </script>}
        
      
### Set Worker Type Field
My last task during the two week sprint was very similar to the user role task in that the admin wanted to be able to update and modify the user's work type from the user list. The only difference being that work type was set as an enum, so the implementation needed to be modified. I created the dropdown like I did with user roles, by starting with Html.BeginForm. But to poulate the dropdown itself, I needed to modify the syntax to accept enum values. 

                @using (Html.BeginForm("EditWork", "Account", FormMethod.Post, new { enctype = "multipart/form-data" }))
                {
                    @Html.Hidden("Id", item.Id)
                    @Html.DropDownList("workType", new SelectList(Enum.GetValues(typeof(ConstructionNew.Enums.WorkType))),                                  item.WorkType.ToString())
                    <input type="submit" value="Submit" onclick="" />
                }
  
This passed the submitted value back to my "EditWork" method as a string called "workType". I then set if statements to match the value of the returned string to the corresponding description of work type. When a match was found, the user's work type was then set to the selected value, stored in the database, and populated properly throughout the program where applicable.

        public ActionResult EditWork(string id, string workType)
        {
            var user = (from ApplicationUser in db.Users where ApplicationUser.Id == id select ApplicationUser).FirstOrDefault();
            if (workType == "Unselected")
            {
                user.WorkType = Enums.WorkType.Unselected;
            }
            if (workType == "LeadMan")
            {
                user.WorkType = Enums.WorkType.LeadMan;
            }
            if (workType == "Foreman")
            {
                user.WorkType = Enums.WorkType.Foreman;
            }
            if (workType == "ExpMBA")
            {
                user.WorkType = Enums.WorkType.ExpMBA;
            }
            if (workType == "NewMBA")
            {
                user.WorkType = Enums.WorkType.NewMBA;
            }

            if (ModelState.IsValid)
            {
                db.Entry(user).State = EntityState.Modified;
                db.SaveChanges();
                return RedirectToAction("Index", "Home");
            }
            return RedirectToAction("Index", "Home");
        }

### Summary
I found this experience to be incredibly valuable to my growth as a developer. Working on a team as opposed to an individual effort creates new challenges, but also opens up the potential for new lessons to be learned. It was interesting to use other people's work as a foundation for my own, and changing or customizing as needed without compromising what had already been established. It was also quite rewarding to see my contributions positively impact other people's work and vice versa. There were a concepts I encountered during this two weeks that I had never seen before; learning them and implementing them was a challenge, yet ultimately an excellent teaching tool. I enjoyed this experience tremendously and truly see the value in working in a team setting. 
