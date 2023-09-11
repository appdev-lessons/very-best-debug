# Very Best Debug

In this project, you will get more practice with GUIs by debugging a Very Best venue tracking app.

Here is your target: [very-best-debug.matchthetarget.com](https://very-best-debug.matchthetarget.com/)

This project includes automated tests, so click on this button to get started:

LTI{Load Very Best Debug assignment}(https://grades.firstdraft.com/launch)[S9ymPy6WCsn18gLbByVbZQ7k]{vfdtzJb5bLYqYwuqgeRKpc5d}(5)[Very Best Debug Project]

Then, fork the repository and create your codespace.

## Workflow

1. The goal is to make the code work by correcting all of the bugs that are contained within
1. Checkout the target to see how your code should function
1. Run `rake sample_data` to populate your database
1. Start with visiting the route `/users`
1. You will see an error message, **Read The Error Message**
1. When you understand the error message, work to figure out a solution to fix the error so the Route/Controller/Action/View functions like the target.
1. Remember that the error will guide you to the bug and our skills as developers are determined by how many bugs we understand how to fix
1. Once you have `/users` working correctly, check the routes file and get each working the same as the target
1. Run `rake grade` once all the routes are working to see what else still does not match the target

### One hint: The POST verb

Any create or update actions should use `post()` routes, and the `<form>` actions associated with them should have a `method="post"`.

---
