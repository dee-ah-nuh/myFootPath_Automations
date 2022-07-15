# ***myFootPath_Automations***

> Close.IO Grab by ID


- Efficently queries on CloseIO for only needed data

- Grabs everything with a Student ID and queries it with the API Key for specific situations

          Code:

          from closeio_api import Client
          import pandas as pd
          import os
          username=os.getlogin()

          #This would allow us to get rid of the predownload from zapier and quickly and efficently query on close for only needed data
          #But its still not faster than just predownloading and grabbing the data there lol so it remains dormant. AKa self building query driver
          def grab(df):
              start=0
              end=200
              rows={}
              df=df.drop_duplicates(subset='EMPLID',keep='first')
              while start<len(df):
                  #This grabs everything with a student id thats in the df youre giving to it
                  #So it'd be a completely clean left join
                  data={#we can only grab 200 things at a time with this call
                  "_limit": 200,
                  "query": {
                      "queries": [
                          {
                              "object_type": "lead",
                              "type": "object_type"
                          },
                          {
                              "queries": [{
                                  "queries":[{
                                                  "condition": {
                                                      "mode": "full_words",
                                                      "type": "term",
                                                      "values": []
                                                  },
                                                  "field": {
                                                      "custom_field_id": "lcf_8xBr0KeZQNBtZMOChusviGn94cJSoiLTaQVDHxNx7PC",
                                                      "type": "custom_field"
                                                  },
                                                  "type": "field_condition"
                                              },                            {
                                          "condition": {
                                              "object_ids": [
                                                  "stat_uEnaaHk6Bz2zkKST0LALLQdC2uPt9MMnl73qoeRBVBM"
                                              ],
                                              "reference_type": "status.lead",
                                              "type": "reference"
                                          },
                                          "field": {
                                              "field_name": "status_id",
                                              "object_type": "lead",
                                              "type": "regular_field"
                                          },
                                          "type": "field_condition"
                                      }],"type":"and"
                                  }
                              ],
                              "type": "and"
                          }
                      ],
                      "type": "and"
                  },
                  '_fields':{"lead":['id','custom']},
                  "sort": []
                  }
                  #Here we grab in range of 200 len
                  for key in df[start:end]['EMPLID']:
                      data['query']['queries'][1]['queries'][0]['queries'][0]['condition']['values'].append(str(key))
                  resp=api.post('data/search',data=data)
                  for lead in resp['data']:
                          #we only care about leadid and custom fields here
                      rows[lead['id']]=lead['custom']
                  #After we use the range we update it to be right
                  start=end
                  end+=200
              #we then transform the grabbed data into a dataframe with the stipulations we require
              df2=pd.DataFrame.from_dict(rows,orient='index')
              df2.reset_index(inplace=True)
              df2 = df2.rename(columns = {'index':'id'})
              print(df2.dtypes)
              return df2

          api = Client('api_26qZCo6enkWewoyUHQvbGd.4PZwmPyklAo0vfrk4yWGrd')
          ##resp=api.post('data/search',data=data)
          ##for lead in resp['data']:
          ##                #we only care about leadid and custom fields here
          ##    rows[lead['id']]=lead['custom']
          df=pd.read_csv(f'C:/Users/{username}/Box Sync/ReEngage NAU Reports/Registrar Reports Automated/Group_Enrolled-2022-07-14 06-04-46 AM.csv')
          df2=grab(df)
          print(df2)
