---
title: تهجير الداتا من MongoDb الى PostgreSQL
layout: single
author_profile: true
header:
  overlay_color: "#000"
  overlay_filter: "0.4"
  overlay_image: "/assets/images/nature-grain.jpg"
  show_overlay_excerpt: false
tags: postgresql mongodb
category: Rails
date: 2021-01-23T01:54:57+03:00
---

## تهجير الداتا من MongoDb الى PostgreSQL

هذه المهم من صعب نسيناها، ما زالت اذكر تفاصليها :اني عملت عليها الامس. لا اعلم السبب بصراحة
ربما لانها مهمة نادرة الحصول مع المبرمجيم بشكل عام ام لانها كانت تمثل تحديا كبيرا لي.
على اي حال احببت ان ادون تفاصيل هذه التجربة لكي لا انساها في المستقبل ولكي يتفيد منها اي مبرمج قد يعمل على مهمة مماثلة لها.


### الاسباب التي ادت الى اتخاذ قرار التجهير
السبب الرئيسي والوحيد لهذا القرار هو ان اختيار قاعدة البيانات MongoDB من البداية  كان قرار غير موفق
تم استخدمها فقط للاستفادة من خاصية الحقل Hash الذي يمكنك من حفظ json داتا

### التحديات التي واجهني

1- اتمام المهمة بدون اي انقطاع بالخدمة
2- تحويل Mongo ObjectID الى UUID


### خطوات التنفيذ

1- لتجاوز مشكلة انقطاع الخدمة كان يتوجب علي تقسيم المهمة الى ثلات مراحل (اصدارات)

####  1-1 الاصدار الاول
1- تنصيب gems المطلوبة

```
#Gemfile
gem 'sinatra-activerecord'
gem 'bson-objectid-to-uuid'
gem 'pg', '~> 0.20.0'
```
التطبيق الذي كنت اعمل عليه كان مبيني على sinatra و ليس rails
لذا محتاج لاضافة `gem 'sinatra-activerecord'` ايضا



2- انشاء postgresql schema

انشاء migration لكل Mongoid::Document ضمن postgresql واستخدام نوع الداتا لكل عامود

مثلا لدي هذا Document 
```
class BannedCustomer
  include Mongoid::Document
  include Mongoid::Timestamps

  field :account_id, type: Integer
  field :notes, type: String
  field :system_banned, type: Boolean, default: false
  field :deleted, type: Boolean, default: false
  
 end
```

سوف تكون migration كالتالي

```
class CreateBannedCustomers < ActiveRecord::Migration
  def change
    create_table :banned_customers, id: :uuid, default: 'gen_random_uuid()' do |t|
      t.integer :account_id
      t.text :notes
      t.boolean :deleted, :default => false
      t.boolean :system_banned, :default => false
      t.timestamps null: false
    end
  end
end

```
3- توليد ActiveRecord models

كل models الخاصة ب Mongo موجودة ضمن المسار التالي 

```
root
├── models
│   └── banned_customers.rb
│   └── another_model.rb
```
لذا نحتاح لاستخدام namespace لكي نميز بين models الخاصة بكل نوع من الداتا بيز

```
root
├── models
│   └── banned_customers.rb
│   └── another_model.rb
│   └── pg (folder)
│      └── banned_customers.rb
│      └── another_model.rb
```

```
module PG
  class BannedCustomer < ActiveRecord::Base
  end
end
```
هكذا يكون لدينا اثنان من models ل BannedCustomer

```
BannedCustomer => MongoDB
PG::BannedCustomer => PostreSQL
```

4- كتابة مهمة `rake task` وظيفتها نقل الدتا من ومزامنتها

```
# tasks/db.rake

namespace :db do
  namespace :sync_postgresql do

    sync_desc = <<-DESC.gsub(/    /, '')
      Sync data between Mongoid and postgresql for specifce Mongoid's model
        $ rake db:sync_postgresql:model MODEL='MyModel'
    DESC
    desc sync_desc
    task :model do
      batch_size = 100
      models = ["BannedCustomer", ....]
      not_synced_ids = []
      last_record_processed = nil
      updated_records_ids = []
      created_records_ids = []

      if ENV['MODEL'].to_s == ''
        puts '='*90, 'USAGE', '='*90, sync_desc, ""
        exit(1)
      else
        if !models.include?(ENV['MODEL'].to_s)
          puts '='*90, 'USAGE', '='*90, sync_desc, ""
          exit(1)
        end
        mongoid_klass  = eval(ENV['MODEL'])
        pg_klass = eval("PG::#{ENV['MODEL'].to_s}")

        after_id = ENV['AFTER_ID'].present? ? ENV['AFTER_ID'].to_s : nil

        if after_id.eql?("0")
          last_record_synced_id = nil
        else
          last_record_synced_id = after_id || PG::MigrationState.find_by(key: "last_#{ENV['MODEL'].to_s.underscore}_id").try(:value)
        end

        puts "last_record_synced_id: #{last_record_synced_id}"

        if last_record_synced_id.present?
          last_record_synced = mongoid_klass.unscoped.find(last_record_synced_id) rescue nil
        end

        last_record_stored = mongoid_klass.unscoped.asc(:created_at).last

        if last_record_synced && (last_record_synced.id == last_record_stored.id)
          puts "All #{ENV['MODEL']} Records Have Been Synced"
        else
          if last_record_synced
            count = mongoid_klass.unscoped.between(created_at: (last_record_synced.created_at..last_record_stored.created_at)).count.to_f
          else
            count = mongoid_klass.unscoped.all.count.to_f
          end

          puts "count: #{count}"

          1.upto((count / batch_size).ceil) do |page|
            if last_record_synced
              @mongodb_records = mongoid_klass.unscoped.between(created_at: (last_record_synced.created_at..last_record_stored.created_at)).page(page).per(batch_size)
            else
              @mongodb_records = mongoid_klass.unscoped.all.page(page).per(batch_size)
            end
            @mongodb_records.each do |record|
              uuid = record._id.to_uuid

              mongo_json = record.to_json
              mongo_attributes = JSON.parse(mongo_json)
              mongo_attributes = mongo_attributes.merge({"id" => uuid})

              mongo_attributes.delete("_id")

              begin
                pg_record = pg_klass.unscoped.find_by(id: uuid)
                if pg_record
                  pg_attributes = pg_record.attributes

                  if pg_attributes != mongo_attributes
                    pg_record.update!(mongo_attributes)
                    updated_records_ids << record._id
                  end
                else
                  pg_klass.create!(mongo_attributes)
                  created_records_ids << record._id
                end
                last_record_processed = record._id
              rescue => e
                Raven.capture_exception(e)
                puts e
                not_synced_ids << record._id
              end
            end

            unless @mongodb_records.empty?
              sync_state = PG::MigrationState.find_or_create_by!(key: "last_#{ENV['MODEL'].to_s.underscore}_id")
              sync_state.update!(value: last_record_processed )
            end

            puts "#{batch_size} #{mongoid_klass} are processed."
          end

          not_synced_state = PG::MigrationState.find_or_create_by!(key: "not_synced_#{ENV['MODEL'].to_s.underscore}_ids")
          not_synced_state_value = not_synced_ids.empty? ? "" : not_synced_ids.join(',')
          not_synced_state.update(value: not_synced_state_value)
        end
      end
    end

    all_sync_desc = <<-DESC.gsub(/    /, '')
      Sync data between Mongoid and postgresql for all Mongoid's models
        $ rake db:sync_postgresql:model:all
    DESC
    desc all_sync_desc
    task :all do
      all_models = ["BannedCustomer", "BannedPaypal", "CreditCardCheck", "PaypalCheck"]
      all_models.each do |klass|
        puts "#" * 40
        puts "Processing model: #{klass}..."
        ENV['MODEL'] = klass.to_s
        Rake::Task["db:sync_postgresql:model"].invoke
        Rake::Task["db:sync_postgresql:model"].reenable
      end
    end
  end
end
```

`PG::MigrationState` هو جدول استخدمته لحفظ حالة التجهير
بالتأكيد يمكن استبداله باي حل آخر متل  in-memory database ولكني بالنسبة الي لم يكن هناك اي خيار آخر متوفر لذا كنت مضطر لحفظها داخل الداتا بيز
5- ابقاء الكتابة والقراءة من MongoDB



بعد تنفيذ الخطوات السابقة تم رفع هذا الاصدار وباليدأ بتنفيذ تاسك المزامنة
ووضعها ضمن cron
بعد 3 ايام تم الـاكد ان لامور كلها تسير كما هو متوقع لذا تم البدأ بالاصدار الثاني

#### 2-1 الاصدار الثاني
في هذا الاصدار تم جعل القراءة والكتابة من PostgreSQL بدلا MongoDb
وتنفيذ مهمة المزامنة للتأكد من مزامنة الداتا


#### 3-1 الاصدار الاخير
تم حذف MongoDb نهائيا بعد التأكد من عمل PostgreSQL  وان جميع الداتا تم نقلها من خلال التاسك السابقة


### الخلاصة
الخطوات السابقة بالرغم من انها قد تبدو بسيطة ولكنها استغرقت وقتًا طويلاً ولكن المهم انها تمت بدون ان يلاحظ المستخدمين اي تغير يذكر. وهذا المطلوب بالتأكيد

أراهن أن هناك العديد من المطورين الذين اضطروا بالفعل للتعامل مع مثل هذه المشكلة، اذا واحد منهم لا تترد بإخباري بتجربتك من خلال التعليقات :)




