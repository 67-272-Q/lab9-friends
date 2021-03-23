#Objectives

- teach students how to send email via Rails
- teach students how to upload files to application
- reinforce earlier Rails development skills
- not continually pester them about Git because they know to commit regularly by now

--

1. We are going to create a very simple Rails project called 'Friends' that will allow us to keep the contact information for our pals.  It should have only one model called `Friend`.  As a reminder, create the Rails app (`rails new Friends`) and use `rails generate scaffold` to create the model.  In terms of attributes, a friend should have a full name (full_name), nickname, email, phone, website and a friendship level.  (Remember to run `rails db:migrate` after creating the scaffolding and don't forget to save your code to your git repo.)  There are 4 friendship levels we will consider: 'Casual', 'Good', 'Close', 'Best'.  Do not make a friendship level model; these levels should be saved as an array in the Friend model as seen below:

  ```ruby
    FRIENDSHIP_LEVELS = [ ["Casual", "1"], ["Good", "2"], ["Close", "3"], ["Best", "4"] ]
  ```

2. Go to the form partial in the `app/views/friends` directory and make the friendship level a select list rather than a text field. Now go to the `Friend` model to add a series of validations.  Make sure that the fields for name, email and level are required.  Using `validates_format_of`, make sure that the phone and email fields only take acceptable values for those attributes.  (Look back at the test arrays for PATS if there is any doubt.)  Finally, the friendship level must be one of the 4 levels allowed; no tampering with the select list should be tolerated.  (If you are confident in your regex skills and have time, add another check that the website is valid.)  Have the TA verify that the basic project is working and all fields are validated appropriately.

- - -
# <span class="mega-icon mega-icon-issue-opened"></span> Stop
Show a TA that you have the Rails app set up and working as instructed and that the code is properly saved to git. Make sure the TA initials your sheet.
- - -

3. In this next step we are going to add a mailer so that our project can send an email every time we add a new friend to our list or delete one.  Switch to a new git branch before going further.  The first step is to set up an initializer with our SMTP settings.  We will use Yahoo Mail in this example, but you can use another service such as Gmail by changing the code below to <code>"smtp.gmail.com"</code> and corresponding Gmail address and password.  Within the config/initializers directory, add a new file called <code>setup_mailer.rb</code> and insert the following code:

  ```ruby
    ActionMailer::Base.smtp_settings = {  
      :address              => "smtp.mail.yahoo.com",       
      :port                 => 587,                         
      :user_name            => "<your Yahoo email address>",  # ENV['YAHOO_ADDR']
      :password             => "<your Yahoo password>",       # ENV['YAHOO_PWD']
      :authentication       => "plain",  
      :enable_starttls_auto => true  
    }
  ```
(**Note:** If you were pushing this lab to a public repo, you would add your username and password as environment variables, or else they would be public to everyone!)

4. Now we want to create a mailer we can use to handle the business of sending mail.  We can create a basic mailer with the following code:

  ```ruby 
    rails generate mailer FriendMailer
  ```

  This will create a new mailers directory with a new `friend_mailer.rb` model for us.  If you haven't saved the code to git yet, do so before we start adding our own code to the mailer.

5. In `application_mailer`, change the default `from` value to your full email address.  In `friend_mailer.rb`, create two methods: `new_friend_msg` and `remove_friend_msg`.  We will pass into the mailer a friend object so we will need to add that argument to each method.  Below is code that I used to set up my first method:

  ```ruby
    def new_friend_msg(friend)
      @friend = friend
      mail(:to => friend.email, :subject => "My New Friend")
   end
  ```

  Adjust your code for both methods accordingly.

6. Now we need to create templates that these methods can call to generate an email body.  Going to the views directory, we see there is a subdirectory for friend_mailer; add two files: `new_friend_msg.text.erb` (for a regular text email) and `remove_friend_msg.html.erb` (for html-formatted email).  We need to be sure to pass in the name of our friend and display it using erb tags.  An example of appropriate templates can be found at the end of the lab; you will want to modify the actual text so that it sounds like it is from you.  (... unless you are also half-Klingon like me, in which case the example templates will work just fine.)

7. Finally, we have to actually send the mail.  There are several ways of doing this, but we will use the most direct.  Go into the `friends_controller.rb` file and look at the `create` method.  After the new friend is created (`if @friend.save`) add the following code:

  ```ruby
    # Provide an email confirmation if all is good...
    FriendMailer.new_friend_msg(@friend).deliver

    # Now a page confirmation as well...
    flash[:notice] = "#{@friend.nickname} has been added as a friend and notified by email."
  ```

  The deliver method will call the new_friend_msg in the FriendMailer model and use the template (having the same name) to send an email to this friend.  Do something similar for the delete method so that a remove_friend_msg is sent when the friend is deleted.

8. Now time for a little testing.  Start up the server (if it's already running you need to restart to get the initializer file to be included) and add a new friend, but be sure to use your email. (Please do not spam anyone during this lab.)  Look at the server output (same window as `rails server` is running in) to see the email was sent.  Now delete that friend and verify the person was notified as well.  (**Note**: If you are using Gmail, you may not see emails in your inbox due to Google's security settings blocking access. An alternative way of displaying the email generation is presented in the next step).

9. You may have noticed that your email did not send ... oh dear. In order to remedy this, we will use the [letter opener gem](https://github.com/ryanb/letter_opener) to save our emails to our app locally. First add the gem to your development environment and run the `bundle` command to install it.

  ```ruby
     gem "letter_opener"
  ```

  Then set the delivery method in `config/environments/development.rb` to 

  ```ruby
    config.action_mailer.delivery_method = :letter_opener
  ```

  Now any email will pop up in your browser instead of being sent. The messages are stored in `tmp/letter_opener`.

  Note: For windows users, the email may not pop up, but you should still be able to find it in `tmp/letter_opener`.

  Now, after confirming email generation by checking your inbox or your browser, merge all this back to the master branch and have a TA verify this with the appropriate check-off.

- - -
# <span class="mega-icon mega-icon-issue-opened"></span> Stop
Show a TA that you have the email functionality set up and working as instructed and that the code is properly saved to git. Make sure the TA initials your sheet.
- - -

10. Now we want to be able to add a photo of our friends so that when we display their listing, a magnificent image appears.  Start by cutting a new branch in git called 'photos'.  To do the uploads we will use a gem known as [CarrierWave](https://github.com/jnicklas/carrierwave).  There are other gems that can also be used – [paperclip](https://github.com/thoughtbot/paperclip) being one of the more popular ones – but carrierwave is easier to set up (IMHO) and works with [ActiveRecord](http://api.rubyonrails.org/classes/ActiveRecord/Base.html), [DataMapper](http://datamapper.org/) and a variety of [ORMs](http://solnic.eu/2011/11/29/the-state-of-ruby-orm.html).  To get this gem installed, go to the Gemfile and type `gem 'carrierwave'` and then `bundle install` to add it to your system gems.

11. Once the plugin is installed, we can run its generator to add a photo attribute to the Friend model with the following line of code:

  ```ruby
    rails generate uploader photo
  ```

  This creates a new directory within app called uploaders and a file within that called  'photo_uploader.rb' which will handle the logic of uploading a file.  Open that file and you will see there are a lot of options commented out for us.  We will leave most of these alone right now (feel free to experiment later), but there is one option we want to uncomment: the `extension_white_list` function. (starts around line 38 or so) This will ensure that the files being uploaded are images; to allow any type of file to be uploaded is a huge security risk and checking the extension types is the least we can do to stop potential abuse of this functionality.

12. The uploader file will not be of much value unless we explicitly connect it to the Friends model and alter the database to record the file path.  First, go into the Friend model and add near the top of the Friend class the following:

  ```ruby
    mount_uploader :photo, PhotoUploader
  ```

  Once the uploader is associated with the Friend model, we will need a migration so the connection can be recorded.  On the command line, type the following:  

  ```ruby
    rails generate migration add_photo_to_friends photo:string
  ```

  Open up this migration and look at the fields that are being created by this operation.  Run `rails db:migrate` to modify the database.

13. Make sure this parameter can be passed along by the controller to the model.  Go to the `friend_params` method in the controller and add :photo to the list of permitted parameters.

14. Now we have to go into the friend form partial and add a field that will allow a user to upload the image.  To start, at the top of the form after the `local: true`, add `multipart: true` so that the form is capable of receiving attachments.  Then go do into the form itself and add the following:

  ```erb
    <div class="field">
      <%= form.label :photo %><br />
      <%= form.file_field :photo %>
    </div>
  ```

  This will create file upload control that will allow users to browse and select a photo to upload.

15. To display a photo that is uploaded, go to the 'show' page for friends and add this code somewhere on the page where it is appropriate:

  ```erb
    <% unless @friend.photo_url.nil? %>
      <p><%= image_tag @friend.photo_url %></p>
    <% end %>
  ```

16. Once this in place, fire up the dev server again and add a photo for your friends.  If you need some appropriately sized photos, samples are available [here](http://pawn.hss.cmu.edu/~profh/friends_pics.zip).  (Warning: if you use these pictures, be sure to make [Darth Vader](http://www.youtube.com/watch?v=UYZoxY3sawE) your friend.  Trust me, [you don't want to hack off the Dark Lord of the Sith](http://www.youtube.com/watch?v=81fwEmP2CKY).)  Make sure you save to git and merge back to the master branch.  Get the TA to sign off and you are done.

- - -
# <span class="mega-icon mega-icon-issue-opened"></span> Stop
Show a TA that you have the completed all portions of the lab, including file upload. Make sure the TA initials your sheet.
- - -

17. **(Optional, on your own)**: If you have time, now you will modify your application to store your friend's photos in the cloud using Amazon Web Services. In this lab, we've so far used your application's file system to accomplish this, but in a production environment like [Heroku](https://www.heroku.com) where we don't have this kind of access, you would need to use cloud storage.  The change is simple, as we'll be configuring our existing carrierwave code to use Amazon S3.

18.  First, you'll need to sign up for an AWS account [here](https://aws.amazon.com/s3/). This account will require a payment method, but the basic version is free of charge (unless you are very popular and have over 10,000 friends). This signup may take some time, and could require a 24 hour activation period (hence, this part of the lab is on your own, though these may be valuable instructions for you to use in future projects).

19.  Once you've signed up for AWS, you need to generate your S3 key and your S3 secret key on your [Security Credentials Page](https://console.aws.amazon.com/iam/home?#/security_credential).  Download the file that has these keys, as you'll need them in a couple of steps.

20.  First, add the following gem to your Gemfile:

  ```ruby
    gem 'fog-aws'
  ```

21.  Next, we'll need to make the following change in `photo_uploader.rb`.  Previously, we were using the file system, but now we want to use fog:

  ```ruby
    # storage :file
    storage :fog
  ```

22.  We also want to make sure we are storing the photos of our friends in the proper bucket on our S3 account. In `photo_uploader.rb`, replace the contents of the `store_dir` function with the following code, making sure to use your Andrew ID or some other unique sequence of characters within the name of the bucket (buckets are uniquely named across all of S3):

  ```ruby
    "<your ANDREW ID>-friends"
  ```

22.  Lastly, we'll need to add a file called `carrierwave.rb` in the `config/initializers/` and configure it to use our key and secret key. Again, be sure to use your Andrew ID or your own naming convention where applicable. (**Note:** If you were pushing this lab to a public repo, you would instead add your keys as environment variables, or else they would be public to everyone!):

  ```ruby
  require 'carrierwave/orm/activerecord'
  CarrierWave.configure do |config|
    config.fog_provider = 'fog/aws'

    # Configuration for Amazon S3 should be made available through an Environment variable
    # For local installations, export the env variable through the shell OR
    # if using Passenger, set an Apache environment variable
    # 
    # In Heroku, follow http://devcenter.heroku.com/articles/config-vars
    # 
    # # $ heroku config:add S3_KEY=your_s3_access_key S3_SECRET=your_s3_secret S3_REGION=eu-west-1 S3_ASSET_URL=http://assets.example.com/ S3_BUCKET_NAME=s3_bucket/folder


    # # Use local storage if in development or test
    # if Rails.env.development? || Rails.env.test?
    #   CarrierWave.configure do |config|
    #     config.storage = :file
    #   end
    # end

    # # Use AWS storage if in production
    # if Rails.env.production?
    #   CarrierWave.configure do |config|
    #     config.storage = :fog
    #   end
    # end
  

    config.fog_credentials = {
      :provider               => 'AWS',                                       # required
      :aws_access_key_id      => '<your S3 key>',                             # ENV['S3_KEY'],
      :aws_secret_access_key  => '<your S3 secret key>',                      # ENV['S3_SECRET']
      :region                 => 'us-east-1'                                  # ENV['S3_REGION']
    }
    config.fog_directory = '<your ANDREW ID>-friends'   # ENV['S3_BUCKET_NAME']

    # "#{ENV['S3_ASSET_URL']}/#{ENV['S3_BUCKET_NAME']}"
    #config.fog_host       = 'https://assets.example.com'          # optional, defaults to nil
    config.fog_public     = false                                  # optional, defaults to true
    config.fog_attributes = {}  # optional, defaults to {}
end
  ```

23.  Let's try it out. Fire up `rails server`, and upload a file by running your project locally. Verify on your Amazon S3 account that this file is in the `<your ANDREW ID>-friends` bucket.

  And that's it! You've just successfully configured your project to use Amazon S3. This was a basic tutorial, but you can read more on the [carrierwave documentation](https://github.com/carrierwaveuploader/carrierwave) under the heading "Using Amazon S3". 

Template Samples
================

**Example of new_friend_msg template:**

  ```erb
    nuqneH <%= @friend.nickname %>,

    I've just listed you as one of my friends.  I am sure you are deeply honored, as you should be.  

    Remember the Klingon proverb regarding friends:

       may'Daq jaHDI' SuvwI' juppu'Daj lonbe' 
       (When a warrior goes to a battle, he does not abandon his friends.)

    Don't forget about me when you go into battle -- I will not forget you.

    Qapla'

    Klingon Code Warrior
  ```

**Example of remove_friend_msg template:**
  ```erb
    <p><%= @friend.full_name %>,</p>

    <p>I had previously listed you as a friend; however, I am reminded of the old Klingon proverb:<br />

      QaghmeylIj tIchID, yIyoH <br />
      <em>(Have the courage to admit your mistakes.)</em><br />
    </p>

    <p>Clearly it was a mistake to make you a friend.  I admit this error in judgment and hereby remove you from my friend list.</p>

    <p>Dajonlu'pa' bIHeghjaj*</p>

    <p>Klingon Code Warrior</p>
    <hr />
    <p><em>*May you die before you are captured.</em></p>