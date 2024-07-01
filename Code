const UssdMenu = require('ussd-menu-builder');
const Papa = require('papaparse');
const fs = require('fs');
const os = require('os');
const path = require('path');
const aws = require('aws-sdk');
aws.config.update({region: 'eu-west-1'});

//Define the provinces. 
const provinces = [ 'Gauteng', 'Western Cape', 'Northern Cape', 'North West', 'Mpumalanga', 'Limpopo', 'KZN', 'Eastern Cape', 'Free State'];
var province;
var npsExit;
var service;
var communication;
var turnaroundTime;
var comment;
var ourSessionId;
var myPhoneNumber;


//Instantiate a new aws S3 object and set Date to create a new date object with the current date and time
const s3 = new aws.S3();
const filePath = path.join(os.tmpdir(), 'temp.csv');

const exportDataToCSV = (data) => {
    // Convert the data to CSV format using PapaParse
    const csv = Papa.unparse(data);
    return csv;
  };

  //Function to upload file to S3
const uploadFileIntoS3 = async () => {
  var fileName = "ussdForm"+ourSessionId+".csv";
  const params = {
        Bucket: "my-ussd-data-bucket3",
        Key: `${fileName}`,
        Body: fs.createReadStream(filePath)
    };

    try {
        // Upload file to S3
        const data = await s3.upload(params).promise();
        console.log(`File uploaded successfully: ${data.Location}`);
        
    } catch (error) {
        console.log(error);
    }
};

const submitCSV = async () => {
    //Store form data as a JSON variable
    const mydata = [
      { Timestamp: new Date().toLocaleString('sv-SE', { timeZone: 'Africa/Johannesburg'})  ,  "Data collection method:":"USSD", cellNumber:myPhoneNumber, "Please select the province where you currently reside:" :province /* province:province */ , "On a scale of 1 to 10, how satisfied are you with the service provided by the staff?" :service, "On a scale of 1 to 10, how satisfied are you with the level of communication?" :communication, "On a scale of 1 to 10, could you rate the turnaround time from when you requested assistance to when you were serviced?" :turnaroundTime, "(Optional)Could you describe your experience with our recent interaction or visit to GPAA regarding your GEPF query?" :comment}
    ];

    //Pass JSON file to Papaparse
    var myCsvData= exportDataToCSV(mydata);
    
    //Convert blob to File
    fs.writeFileSync(filePath, myCsvData, function (err) {
      if (err) {
          context.fail("writeFile failed: " + err);
      } else {
        context.succeed("writeFile succeeded");
      }
    });
    await uploadFileIntoS3();
   
  };

exports.handler = (event, context, callback) => {
   
    var menu = new UssdMenu();
    ourSessionId = event.sessionId;
    myPhoneNumber = event.phoneNumber;
    
let args = {
        phoneNumber: event.phoneNumber,
        sessionId: event.sessionId,
        serviceCode: event.serviceCode,
        text: event.text
    };
// Define menu states
    let startState = {
        run: () => {
            // use menu.con() to send response without terminating session
            menu.con('Welcome to the GEPF NPS survey, where your voice matters. Please help us serve you better.' + 
                '\n--------------' +
                '\n1. Continue to NPS Survey' +
                '\n0. Exit');
        },
        // next object links to next state based on user input
        next: {
            '1': 'Continue to NPS Survey',
            '0': 'Exit'
            
        }
        
    };
    
    menu.startState(startState);
    menu.state('start', startState);
    
    menu.state('Continue to NPS Survey', {
        run: () => {
            // use menu.con() to send response without terminating session
            npsExit = menu.val;
            menu.con('Which Province do you reside in'  +
              '\n1. ' + provinces[0] +
              '\n2. ' + provinces[1] +
              '\n3. ' + provinces[2] +
              '\n4. ' + provinces[3] +
              '\n5. ' + provinces[4] +
              '\n6. ' + provinces[5] +
              '\n7. ' + provinces[6] +
              '\n8. ' + provinces[7] +
              '\n9. ' + provinces[8]
            );
            
        },
        next: {
             '1': 'Question 1',
             '2': 'Question 1',
             '3': 'Question 1',
             '4': 'Question 1',
             '5': 'Question 1',
             '6': 'Question 1',
             '7': 'Question 1',
             '8': 'Question 1',
             '9': 'Question 1'
        }
       
        },
        
    );
    
    
    menu.state('Question 1', {
        run: () => {
            
            menu.con('On a scale of 1 to 10, how satisfied are you with the service provided by the staff?');
            province = provinces[parseInt(menu.val, 10) - 1];
        }, 
        next: {
             '1': 'Question 2',
             '2': 'Question 2',
             '3': 'Question 2',
             '4': 'Question 2',
             '5': 'Question 2',
             '6': 'Question 2',
             '7': 'Question 2',
             '8': 'Question 2',
             '9': 'Question 2',
             '10': 'Question 2'
        },
    }
   
    );
    
     menu.state('Question 2', {
        run: () => {
            menu.con('On a scale of 1 to 10, how satisfied are you with the level of communication?');
            service = menu.val;
        }, 
        next: {
             '1': 'Question 3',
             '2': 'Question 3',
             '3': 'Question 3',
             '4': 'Question 3',
             '5': 'Question 3',
             '6': 'Question 3',
             '7': 'Question 3',
             '8': 'Question 3',
             '9': 'Question 3',
             '10': 'Question 3'
        }
    }
   
    );
    
     menu.state('Question 3', {
        run: () => {
            menu.con('On a scale of 1 to 10, could you rate the turnaround time from when you requested assistance to when you were serviced?');
            communication = menu.val;
        }, 
        next: {
             '1': 'Question 4',
             '2': 'Question 4',
             '3': 'Question 4',
             '4': 'Question 4',
             '5': 'Question 4',
             '6': 'Question 4',
             '7': 'Question 4',
             '8': 'Question 4',
             '9': 'Question 4',
             '10': 'Question 4'
        }
    }
   
    );
    
      menu.state('Question 4', {
        run: () => {
            menu.con('Please describe your experience with GPAA regarding your GEPF query?' + ' \n' + '1. Exit');
            turnaroundTime = menu.val;
        }, 
        next: {
            '1': 'Comment',
             '*': 'Exit',
        }
    }
   
    );
    
    menu.state('Comment', {
        run: () => {
            comment = String("No comment");
            menu.end('Thank you for participating. Your feedback is appreciated. Goodbye.');
            submitCSV();
        }
    });
    
    menu.state('Exit', {
        run: () => {
            comment = String(menu.val);
            menu.end('Thank you for participating. Your feedback is appreciated. Goodbye.');
            submitCSV();
        }
    });
    
menu.on('error', (err) => {
    // handle errors
    console.log('Error', err);
});

menu.run(args, resMsg => {
        console.log(resMsg);
        console.log(`Timestamp: ${new Date()}`);
        callback(null, resMsg);
    });
    
};
