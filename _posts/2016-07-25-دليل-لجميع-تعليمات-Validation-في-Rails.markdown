---
title: دليل لجميع تعليمات Validation في Rails
date: 2016-07-25T16:12:05+03:00
layout: single
author_profile: true
header:
  overlay_color: "#000"
  overlay_filter: "0.4"
  overlay_image: "/assets/images/nature-grain.jpg"
  show_overlay_excerpt: false
tags: Validation
category: Rails
---

~~~ruby
class SomeClass < ActiveRecord::Base

  # length
  validates :content, :length => {
    :minimum   => 300,
    :maximum   => 400,
    :tokenizer => lambda { |str| str.scan(/\w+/) },
    :too_short => "must have at least %{count} words",
    :too_long  => "must have at most %{count} words"
  }
  validates :name,
    :presence => true,
    :length => { :within => 1..255, :allow_blank => true }
	
  # format
  validates :legacy_code, :format => { :with => /\A[a-zA-Z]+\z/,
    :message => "Only letters allowed" }

  validates_format_of :email, 
    :with => /\A([^@\s]+)@((?:[-a-z0-9]+\.)+[a-z]{2,})\z/i ,
    :message => "E-mail is incorrect"

  # in
  validates :size, :inclusion => { :in => %w(small medium large),
    :message => "%{value} is not a valid size" }

  # numericality
  validates :points, :numericality => true
  validates :games_played, :numericality => { :only_integer => true }    # Uses regex /\A[+-]?\d+\Z/

  # presence
  validates :order_id, :presence => true
  validates_presence_of :order_
  # uniqueness
  validates :email, :uniqueness => true
  validates :name, :uniqueness => { :scope => :year, :message => "should happen once per year" }
  validates :name, :uniqueness => { :case_sensitive => false }


  # allow_blank
  validates :title, :length => { :is => 5 }, :allow_blank => true

  # allow_nil
  validates :size, :inclusion => { :in => %w(small medium large),
    :message => "%{value} is not a valid size" }, :allow_nil => true

  # on
  validates :email, :uniqueness => true, :on => :create

  # conditionals
  validates :card_number, :presence => true, :if => :paid_with_card?

end  
~~~