---
title: القاسم المشترك الأعظم  Greatest common divisor
date: 2016-07-11T12:48:07+03:00
layout: single
author_profile: true
header:
  overlay_color: "#000"
  overlay_filter: "0.4"
  overlay_image: nature-grain.jpg
tags: GCD
category: Algorithms
---

### طرق حساب GCD

 * الطريقة شائعة لحساب القاسم المشترك الأعظم هو إيجاد العوامل الأولية لكل من العددين ثم اختار العوامل المشتركة بين الاثنين

* الطريقة الآخرى هي خوارزمية اقليدس  

** خطوات هذه الخوارزمية: **

 1- تقسيم العدد الكبير وليكن  A على العدد الصغير وليكن B

 2- فحص ناتج عملية القسمة R، إذا كان يساوي الصفر  فالقاسم المشترك الأعظم هو B

 3- إذا كان R لا يساوي الصفر، نجعل A يساوي B و B يساوي R ونعيد الخطوة الأولى.


~~~ruby
class GCD
	def initialize m,n 
		@m= m
		@n =n
		puts " M | N | R "
		puts " #{@m} | #{@n} |  "
		test_number
	end

	def lookup_for_gcd
		if @x% @y == 0
		    puts " #{@x} | #{@y} | #{@x % @y} "	
		    puts "GCD: #{@y}"
		else
		    r = @x % @y
		    print(@x,@y,r)
		    @x = @y
		    @y = r
		    return lookup_for_gcd
		end
	end

	private

	def print(a,b,r =nil)
		puts " #{a} | #{b} | #{r} "
	end


	def test_number
		if  @m < 0 or @n < 0
			raise 'one of number in negative'
		elsif @m < @n
			print(@x,@y)
		end
		@x =[@m,@n].max
		@y =[@m,@n].min
	end
end

puts "Greatest common divisor "

print "please enter the first number: "
num1 = gets.to_i

print "please enter the second number: "
num2 = gets.to_i

gcd =GCD.new(num1,num2).lookup_for_gcd
~~~