---
title: المصادقة عبر مواقع التواصل الاجتماعي باستخدام OmniAuth + Devise
layout: single
author_profile: true
header:
  overlay_color: "#000"
  overlay_filter: "0.4"
  overlay_image: "/assets/images/nature-grain.jpg"
  show_overlay_excerpt: false
tags: devise
category: Rails
date: 2016-04-05T01:20:44+03:00
---

الهدف من هذه التدوينة هو توضيح كيفية إنشاء  `omniauth - multiple provider` عبر devise
بحيث يتم التحقق (باستخدام email) من وجود حساب مسجل مسبقا ضمن database  وإنشاء userprvider خاص به بدلا من إنشاء حساب جديد  

بما ان  twitter لا يعرض email عبر api  وجهت المستخدم لإدخال البريد الخاص به والتحقق منه قبل إنشاء حساب جديد.



1- إضافة الحزم التالية إلى ملف `Gemfile`

~~~
gem 'omniauth'
gem 'omniauth-twitter'
gem 'omniauth-facebook'
gem 'omniauth-linkedin'
gem 'omniauth-github'
~~~

بعدها تنفيذ الأمر `bundle install`  

2- إنشاء Model   `Userprovider` 


    rails g model userprovider previder:string uid:string user:references:index
    rake db:migrate

`app/model/userprovider.rb`

~~~ruby
class Userprovider < ActiveRecord::Base
	belongs_to :user


  def self.find_for_facebook_oauth(auth)
  	user = Userprovider.where(:provider => auth.provider, :uid => auth.uid).first
        unless user.nil?
            user.user
        else
            registered_user = User.where(:email => auth.info.email).first
            unless registered_user.nil?
                Userprovider.create!(
                    provider: auth.provider,
                    uid: auth.uid,
                    user_id: registered_user.id
                    )
                registered_user
            else
                user = User.create!(
                    name: auth.info.name,
                    email: auth.info.email,
                    password: Devise.friendly_token[4, 30],
                    )
                user_provider = Userprovider.create!(
                    provider:auth.provider,
                    uid:auth.uid,
                    user_id: user.id
                    )
                user
            end
        end
  end

  def self.find_for_twitter_oauth(auth)
    user = Userprovider.where(:provider => auth.provider, :uid => auth.uid).first
    unless user.nil?
        user.user
    else
        false
    end
  end  
end


~~~

3- التصريح عن المزود/(الشبكة الاجتماعية) المستخدم ضمن ملف `config/initializers/devise.rb:` 
 
    config.omniauth :facebook, "APP_ID", "APP_SECRET"
    config.omniauth :twitter, "APP_ID", "APP_SECRET"
    config.omniauth :github, "APP_ID", "APP_SECRET"
    ...


4- إضافة الوحدة `omniauthable` إلى devise

    devise :omniauthable, :omniauth_providers => [:facebook,:twitter,:github]

`app/model/user.rb`

~~~ruby
class User < ActiveRecord::Base
	

  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :trackable, :validatable,:omniauthable, :omniauth_providers => [:facebook,:twitter]

  has_many :userprovisers , :dependent => :destroy

end

~~~

5- إضافة controller `app/controllers/users/omniauth_callbacks_controller.rb` 

~~~ruby
class Users::OmniauthCallbacksController < Devise::OmniauthCallbacksController
  
  def facebook
    provider = "facebook"
    @user = Userprovider.find_for_facebook_oauth(request.env['omniauth.auth'])
      if @user.persisted?
        flash[:notice] = I18n.t('devise.omniauth_callbacks.success', kind: provider.capitalize)
        sign_in_and_redirect @user, event: :authentication
      else
        session["devise.facebook_data"] = request.env["omniauth.auth"]
        redirect_to new_user_registration_url
      end
  end

  def twitter
    provider = "twitter"
    auth = request.env['omniauth.auth']
    @user = Userprovider.find_for_twitter_oauth(request.env['omniauth.auth'])
      if @user.persisted?
          if @user.email.split("@").last == "example.com"
              sign_in  @user, event: :authentication
              render :add_email
          else
              sign_in_and_redirect  @user, event: :authentication   
              flash[:notice] = I18n.t('devise.omniauth_callbacks.success', kind: provider.capitalize)
          end
      else
        session["devise.twitter_data"] = request.env["omniauth.auth"].except("extra")
        redirect_to new_user_registration_url
      end
  end

end

~~~
6- إضافة Controller  لأجل حفظ البريد الالكتروني عند استخدام Twitter

`app/controllers/users/accounts_controller.rb`



~~~ruby
class Users::AccountsController < ApplicationController

  def update
    auth =  session[:hash]
    registered_user = User.find_by_email(params[:email])
      unless registered_user.nil?
        Userprovider.create!(
          provider: auth["provider"],
          uid: auth["uid"],
          user_id: registered_user.id
              )
          flash[:notice] = I18n.t "devise.registrations.signed_up"
          sign_in_and_redirect registered_user, event: :authentication
          
      else
       user = User.create!(
          name: auth["info"]["name"],
          email: params[:email],
          password: Devise.friendly_token[4, 30],
          )
        Userprovider.create!(
          provider:auth["provider"],
          uid:auth["uid"],
          user_id: user.id
          )
          flash[:notice] = I18n.t "devise.registrations.signed_up"
          sign_in_and_redirect user, event: :authentication
      end
  end

end

~~~

7 - إضافة  view لإضافة البريد `app/views/users/omniauth_callbacks/add_email.html.erb`


    <%= form_tag(account_update_path,:method => :put) do%>
      <div class="field">
          <label>البريد الالكتروني:</label>
    		    <%= email_field_tag :email %>
      </div>
      <br>
      <div class="field"> 
    		<%= submit_tag "اكمل التسجيل ", :class => "button color" %>
      </div>
    <% end %>


8- إضافة  routes الضرورية

`routes.rb`

~~~ruby
  devise_for :users, :controllers => { :omniauth_callbacks => "users/omniauth_callbacks" }
  devise_scope :user do
    put "account/update" => "users/accounts#update" , as: :account_update
    patch "account/update" => "users/accounts#update"
  end
~~~
    

9- أخيرا رابط التسجيل يكون كالتالي:

    <%= link_to "Sign in with Facebook", user_omniauth_authorize_path(:facebook) %>


المشكلة الموجود بالطريقة السابقة هو ان المستخدم إذا كان مسجل بشكل يدوي ببريد الكتروني وقام بتسجيل عبر توتير يتم إنشاء حساب جديد لعدم وجود إمكانية للتحقق من email









