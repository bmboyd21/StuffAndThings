import os
from flask import Flask, request, jsonify, send_file 
import json
import requests
import datetime

app = Flask(__name__)
port = int(os.environ.get('PORT', 3000))


@app.route('/check_number', methods=['POST']) 
def check_number_validation(): 

  bot_data = json.loads(request.get_data())
  parcel_number = bot_data['conversation']['memory']['parcel-number']['raw']
  memory = bot_data['conversation']['memory']

  if len(parcel_number) == 18:
    validation = 1
  else:
    validation = 0

  memory['validation'] = validation

  return jsonify(
    status=200,
    conversation={
      'memory': memory
    }
  )

@app.route('/track_parcel', methods=['POST']) 
def get_track_parcel(): 
  
  bot_data = json.loads(request.get_data())
  parcel_number = bot_data['conversation']['memory']['parcel-number']['raw']

  json_requests = {
    "UPSSecurity": {
      "UsernameToken": {
        "Username": "ysu_deliverybot",
        "Password": "190819&Jiqiren4TEST" },
      "ServiceAccessToken": {
        "AccessLicenseNumber": "0D6985C32BF33012"
      } },
    "TrackRequest": {
      "Request": {
        "RequestOption": "1", 
        "TransactionReference": {
          "CustomerContext": "Your Test Case Summary Description" }
      },
      "InquiryNumber": parcel_number }
      #1Z12345E6605272234 : une seule réponse
      #1Z12345E0205271688 : plusieurs réponse
      #1ZISDE016691609089 : pas de réponse
  }
  url = "https://wwwcie.ups.com/rest/Track"
  r = requests.post(url, json=json_requests)

  data = r.json()

  if 'TrackResponse' in data.keys():
    if type(data['TrackResponse']['Shipment']['Package']['Activity'])  == list:
      activity = data['TrackResponse']['Shipment']['Package']['Activity'][0]
    else:
      activity = data['TrackResponse']['Shipment']['Package']['Activity']
    date_str = activity['Date']
    date_obj = datetime.datetime.strptime(date_str, '%Y%m%d')
    RES = 'Latest status of your parcel: ' + activity['Status']['Description'] + ' on ' + date_obj.strftime('%B %-d, %Y')
  else:
    RES = 'Sorry I could not find any information with the offered number. I met the error: ' + data['Fault']['detail']['Errors']['ErrorDetail']['PrimaryErrorCode']['Description']

  return jsonify( 
    status=200, 
    replies=[
    {
      'type': 'text', 
      'content': RES,
    },]
  )
  #data['TrackResponse']['Shipment']['Package']['Activity']['Status']
  #No tracking information available
  #{"Fault":{"detail":{"Errors":{"ErrorDetail":{"PrimaryErrorCode":{"Code":"151044","Description":"No tracking information available"},"Severity":"Hard"}}},"faultcode":"Client","faultstring":"An exception has been raised as a result of client data."}}

@app.route('/save_size', methods=['POST']) 
def save_parcel_size(): 

  print("save parcel size")
  bot_data = json.loads(request.get_data())
  if 'parcel-size' in bot_data['nlp']['entities'].keys():
      memory = bot_data['conversation']['memory']
      print(bot_data['nlp']['entities']['parcel-size'][0])
      memory['parcel-size'] = bot_data['nlp']['entities']['parcel-size'][0]
      return jsonify(
        status=200,
        conversation={
          'memory': memory
        }
      )
  return jsonify(status=200)

@app.route('/get_location_thumbnail', methods=['POST'])
def get_location_thumbnail():
  bot_data = json.loads(request.get_data())
  geolocation = bot_data['conversation']['memory']['location']['formatted']
  # Python program to get a google map 
  # image of specified location using 
  # Google Static Maps API 

  # Enter your Google Map api key here 
  api_key = "xxx"

  # url variable store url 
  url = "https://maps.googleapis.com/maps/api/staticmap?"

  # center defines the center of the map, 
  # equidistant from all edges of the map. 
  center = geolocation

  # zoom defines the zoom 
  # level of the map 
  zoom = 14

  # get method of requests module 
  # return response object 
  r = requests.get(url + "center=" + center + "&zoom=" +
          str(zoom) + "&size=400x400&key=" +
                api_key + "&sensor=false") 

  # wb mode is stand for write binary mode 
  filePath = "images/" + center + ".jpg"
  f = open(filePath, 'wb') 

  # r.content gives content, 
  # in this case gives image 
  f.write(r.content) 

  # close method of file object 
  # save and close the file 
  f.close() 

  return jsonify( 
    status=200, 
    replies=[
    {
      'type': 'picture', 
      'content': 'https://sapupschatbot.cfapps.eu10.hana.ondemand.com/' + filePath,
    }]
  )

@app.route('/images/')
def get_image(imageName):
  return send_file('images/' + imageName, mimetype='image/jpg')


@app.route('/errors', methods=['POST']) 
def errors(): 
  print(json.loads(request.get_data())) 
  return jsonify(status=200)

@app.route('/')
def hello():
  return "UPS Chatbot running!"

if __name__ == '__main__':
  app.run(host='0.0.0.0', port=port)
