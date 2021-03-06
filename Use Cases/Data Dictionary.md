
# Creating a Data Dictionary

### Step 1: Build out LookML Model
Within your LookML Model, define Fields with descriptions, labels, and other parameters that you might to pull into your data dictionary.

### Step 2: Use API call to pull in LookML Metadata
Connect to our API and call the following Method (lookML_model_explore) to pull in the appropriate fields that you would want to identify within your Data Dictionary. 

Sample Ruby code can be found below.

```
# Get Data Dictionary 

require 'looker-sdk'

module ApplicationHelper

	def self.api_auth
		sdk = LookerSDK::Client.new(
			# Looker/API Credentials
			:client_id => ENV['API_CLIENT_ID'],
			:client_secret => ENV['API_SECRET'],
			:api_endpoint => ENV['API_ENDPOINT'],
			:connection_options => {:ssl => {:verify => false}}
		)
		return sdk
	end

	def self.get_field_values(model_name, explore_name)

		sdk = self.api_auth()
		fields = {:fields => 'id, name, description, fields'}

		#API Call to pull in metadata about fields in a particular explore
		fields = sdk.lookml_model_explore(model_name, explore_name, fields)


		my_fields = []

		#Iterate through the field definitions and pull in the description, sql, and other looker tags you might want to include in  your data dictionary. 
		fields[:fields][:dimensions].to_a.each do |x|
			dimension = {
				:field_type => 'Dimension',
				:view_name => x[:view_label].to_s,
				:field_name => x[:label_short].to_s, 
				:type => x[:type].to_s,
				:description => x[:description].to_s,
				:sql => x[:sql].to_s
			}
			my_fields << dimension
		end

		fields[:fields][:measures].to_a.each do |x|
			measure = {
				:field_type => 'Measure',
				:view_name => x[:view_label].to_s,
				:field_name => x[:label_short].to_s, 
				:type => x[:type].to_s,
				:description => x[:description].to_s,
				:sql => x[:sql].to_s
			}

			my_fields << measure
		end

		return my_fields
	end

end
```

### Step 3:

Format the results from the lookML_model_explore call using custom CSS and HTML. Consider using JS Plugins to allow for Search and Sort functionality to make the dictionary more accessible. In this Block, we use the [DataTable JS Plugin](https://datatables.net/).
