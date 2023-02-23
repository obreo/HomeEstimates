# HomeEstimates
#### Video Demo:  <https://youtu.be/3mKH6z7zUOg>
#### Description:
#### HomeEstimates is a platform that delivers Construction Quantity estimation services for both client and quantity surveyors. These services include calculation tools, Sending an Estimation Query by hiring an expert, and Working in the platform as freelancers.

#### The Platform contains five sections:
> 1. Homepage:          provides a quantity estimation calculator and offers the above-mentioned services.
> 2. Client:       in which he sends a query.
> 3. Freelancers:   in which they register themselves on the platform.
> 4. Shopping:     To buy items provided by the platform.
> 5. About

This platform works as a Flask application.

## How It's Done

Every section has its route in flask. Starting with each section individually:

### 1. Homepage
First, we will create a python app called **app.py** and we will call Flask library:
> from flask import Flask, render_template, request, session, redirect, flash

Now make a route called **index** and let it return the index.html page which we will create in the next step:
> @app.route("/")
def index():
    return render_template("index.html")

Before creating the index.html file, we need to make a layout.html file that will design the main static elements of all pages and extend it in the other HTML files.

Such as the navigation bar, the logo, and the footer. Then inside the main element, create a **{% block main %}{% endblock %}** block to be used in the rest of the pages.

Now, create an HTML page called index and make sure it resides in the Templates folder including the layout.html file in the flask app folder. The HTML page will be the welcoming page.

### Excel sheet
HomeEstimates platform uses an Embedded Excel sheet calculator. The excel sheet will be prepared in a sense to be protected from the Microsoft Excel software and then uploaded to Onedrive - Microsoft's cloud storage platform. After uploading, right-click the excel file > embed > generate an embed link. A URL link guides to Microsoft suite 360 will pop up, click on it and from there choose the **Javascript** embed link.

#### Note: You can control your sheet from the Above mentioned link and choose how it should behave on websites.

Copy the Javascript syntax and paste it into the index.html and it should show up on the homepage once you test it. You may modify the width and height of the Excel sheet window from the width & height attributes in the <script> tag.

### 2. hire page
This is the client's page, I created a form in which the client would fill out his query and attach supporting documents. The query will be sent through an email to my assigned email containing the uploaded attachments. For this, we will create work.html and link it to the **/work** route in **app.py**.

#### a. hire.html:
First of all, create a form attribute under the POST method having a name, email, title, and inbox blocks. also include an **< input > type = file** for the attachments and a submit button at the end of the form. We need to name every element in the HTML page with the **name** tag so we can call the data inserted in these elements by the user.
> name = name | name = email | ..etc

For the attachment element; input type file, we need to add extra tags at the form attribute to be able to call the attached files:
> enctype="multipart/form-data"

If the elements aren't filled, then we will return the hire.html page with an error string, else it will proceed with a success string.

#### b. @app.route("/hire")
Create a /hire route in the app.py file and put an exception to use the POST method. Now we will call each element by the name tag we used:
> name = request.form.get("name")..etc

However, for the attachment element, we will use **request.files("")**. We need this data to forward it to our email, which uses Flask mail.

### b.1- Flask mail

Call the flask mail library and set the configurations required for the email you want to use to send the HTML data:
> from flask_mail import Mail, Message

Make sure that the email you are using can be accessed from your machine's IP address. This is done by modifying the security in your email account settings. Now set the flask mail message code as mentioned in the official documentation.
```
msg = Message(title, sender = 'EMAIL_ADDRESS', recipients = ['EMAIL_ADDRESS'])

msg.body = email + "\n\n" + name + "\n\n" + query
```

The sender's email will be the same as the one you have set in the configurations. the receiver will be your secondary email in which you want to receive the query. the title will use the same title as well.
The **msg.body** will include the content that will be sent in the email, so I have included the user's email, name, and query data inputs that we called from the HTML page, so write the variables instead.

However, for the attachment file, we need to apply extra steps before attaching them to the email.

#### b.2- Attachement <input type=file>
When the client attaches a file, he is uploading it from his machine to the server. This means it takes place - being saved - in the server in which it will be uploaded. So, we have to define a folder in which it will be saved then we can attach it to the email and send it:
```
#Save Uploaded file
if file:
            filename = secure_filename(file.filename)
            file.save(os.path.join(app.config['UPLOAD_FOLDER'] ,filename))
```
After saving the attached file, we will combine the attachment file in the flask mail code:
```
# Attach the Uploaded file with Flask Mail
with app.open_resource((f"/DIRECTORY/{filename}"),'rb') as fp:
        msg.attach(filename, "application/pdf" , fp.read())

# Send Message
mail.send(msg)

# Remove the uploaded file
os.remove((f"/DIRECTORY/{filename}"))
```

### 3. Work page

The Work page is the place where a freelancer would fill his query. It's simply an email list page in which the freelancer would register his name and email address. So whenever a client sends a query, it will be forwarded to the freelancers' email list as a promotion.

For this,s we need to use Flask mail and an email list tool's API, such MailChimp.

#### a. Work.html:
Create an HTML page that e directs to the /hire route having a form of three inputs: name, email, and message. Each will have a name tag to be called in the flask app.

#### b. @app.route("/work")

Create a work route under the POST method. First, we will call the data registered in the HTML elements as part two. Then we will use flask mail again to send the data from the primary email to the secondary email. Then the freelancer's email address and name will be recorded in the email list created on the Mailchimp account.

#### b.1. MailChimp configuration

>A. Create a Mailchimp account.

>B. Create a Campaign called Freelancers.

>C. Go to Account settings > Billing > API, and enable API keys. You'll get your API Key which we will use in the project.

>D. Download the Mailchimp python library.

You can refer to the following documentation to learn more about how to use the Mailchimp API <https://pypi.org/project/mailchimp3/1.0.17/>

#### b.2. Setting up Mailchimp

Call the Mailchimp library
>from mailchimp3 import MailChimp

Now, add the API Key
>client = MailChimp('YOUR USERNAME', 'YOUR SECRET KEY')

Then we will simply use the code as mentioned in the documentation where we will assign the name and email in the list. But first, we need to know our list id number, so let's test our Mailchimp data in a separate .py app:

a. Create a new python file and add the Mailchimp library then use the following code:
>client.list.all()  # returns all the lists

Run the app in the terminal and it shall show you a number that is your campaign's ID. Now go back to the app.py and use this in the code and use the email and name variables that we called from the HTML in the following code:
```
        # add John Doe with email john.doe@example.com to list matching id '123456'
        client.lists.members.create('ID', {
            'email_address': email,
            'status': 'subscribed',
            'merge_fields': {
                'FNAME': name,
                'LNAME': '',
            },
        })
```
The app will return the work.html page with success result.

### @app.route("/signup.html")

The signup page is used to avoid scamming accounts that attempt to sign in the email list for the /work. We will create signup.html having the email, passaword, confrim password, and username inputs. Each of the inputs will be called and will be recorded in SQL database.
The database will have four variables: Id, username, password(encrypted), and email.

### @app.route("/login.html")

The login page will be required for when a user enters the /Work page. So we need to include @app.login_required and define it in the library. Also, we need to define the **session** function in the Flask library which checks the user login.

The session function remembers the username login by linking it with the database.

### @app.route("/logout")
Here we will create a logout button in the layout.html, and use the **If Block** to make a condition for a session:
> If session: show logout buttons.

>Else: show login & signup buttons.
