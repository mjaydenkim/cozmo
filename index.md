

### cozmo
my cozmo project controls cozmo by calling in to the number +1 (925) 29 COZMO.

```markdown
from flask import Flask, request
import cube_stack
from twilio import twiml
from multiprocessing import Process, Value
import sys
import cozmo
import time
from cozmo.util import degrees, distance_mm, speed_mmps
app = Flask(__name__)

@app.route("/voice", methods=['GET', 'POST'])
def voice():
    """Respond to incoming phone calls with a menu of options"""
    # Start our TwiML response
    resp = twiml.Response()

    # Start our <Gather> verb
    with resp.gather(numDigits=1, action='/gather') as gather:
        gather.say('If you would like Cozmo to turn right, press 1. If you would like Cozmo to turn left, press 2. If you would like cozmo to go forward, press 3. If you would like cozmo to go backward, press 4. If you would like Cozmo to stack, press 5. If you would like Cozmo to do a cute animation, press 6. If you would like Cozmo to turn its lights, press 7.')

    # If the user doesn't select an option, redirect them into a loop
    resp.redirect('/voice')

    return str(resp)

@app.route('/gather', methods=['GET', 'POST'])
def gather():
    """Processes results from the <Gather> prompt in /voice"""
    # Start our TwiML response
    resp = twiml.Response()

    # If Twilio's request to our app included already gathered digits,
    # process them
    if 'Digits' in request.values:
        # Get which digit the caller chose
        choice = request.values['Digits']

        # <Say> a different message depending on the caller's choice
        if choice == '1':
            resp.say('You selected right.')
            cozmo_command.value = 1
            
        elif choice == '2':
            resp.say('You selected left.')
            cozmo_command.value = 2
            
        elif choice == '3':
            resp.say('You selected forward.')
            cozmo_command.value = 3
            
        elif choice == '4':
            resp.say("You selected backward.")
            cozmo_command.value = 4
            
        elif choice == '5':
            resp.say("You selected stack blocks.")
            cozmo_command.value = 5
        elif choice == '6':
            resp.say("You selected the best animation ever.")
            cozmo_command.value = 6
        elif choice == '7':
            resp.say("You selected Cozmo's light.")
            cozmo_command.value = 7
            
        else:
            # If the caller didn't choose 1 or 2, apologize and ask them again
            resp.say("Sorry, I don't understand that choice.")

    # If the user didn't choose 1 or 2 (or anything), send them back to /voice
    resp.redirect('/voice')

    return str(resp)





def run_cozmo():

    def run(sdk_conn):
        #The run method runs once Cozmo is connected.
        
        robot = sdk_conn.wait_for_robot()
        robot.say_text("Hello World").wait_for_completed()
        robot.say_text("Hello World Again").wait_for_completed()
        global cozmo_command
        while True:
            if cozmo_command.value == 1:
                robot.turn_in_place(degrees(90)).wait_for_completed()
                cozmo_command.value = 0
                time.sleep(1)
            if cozmo_command.value == 2:
                robot.turn_in_place(degrees(-90)).wait_for_completed()
                cozmo_command.value = 0
            if cozmo_command.value == 3:
                robot.drive_straight(distance_mm(150), speed_mmps(50)).wait_for_completed()
                cozmo_command.value = 0
            if cozmo_command.value == 4:
                robot.drive_straight(distance_mm(-150), speed_mmps(50)).wait_for_completed() 
                cozmo_command.value = 0
            if cozmo_command.value == 5:
                
                lookaround = robot.start_behavior(cozmo.behavior.BehaviorTypes.LookAroundInPlace)
                cubes = robot.world.wait_until_observe_num_objects(num=2, object_type=cozmo.objects.LightCube, timeout=60)
                lookaround.stop()
                if len(cubes) < 2:
                    print("Error: need 2 Cubes but only found", len(cubes), "Cube(s)")
                else:
                    robot.pickup_object(cubes[0]).wait_for_completed()
                    robot.place_on_object(cubes[1]).wait_for_completed()
            if cozmo_command.value == 6:
                anim = robot.play_anim_trigger(cozmo.anim.Triggers.MajorWin)
                anim.wait_for_completed()
                cozmo_command.value = 0
            if cozmo_command.value == 7:
                robot.set_all_backpack_lights(cozmo.lights.green_light)
                time.sleep(1)
                robot.set_all_backpack_lights(cozmo.lights.red_light)
                time.sleep(1)
                robot.set_all_backpack_lights(cozmo.lights.blue_light)
                time.sleep(1)
                robot.set_all_backpack_lights(cozmo.lights.white_light)
                time.sleep(1)
                robot.set_all_backpack_lights(cozmo.lights.off_light)
                
                   
    cozmo.setup_basic_logging()
    try:
        cozmo.connect(run)
    except cozmo.ConnectionError as e:
        sys.exit("A connection error occurred: %s" % e)                   

if __name__ == "__main__":

    cozmo_command = Value('i', 0)
    p1 = Process(target = run_cozmo, args=())
    p1.start()
    p2 = Process(target = app.run(debug=True))
    p2.start()
    p1.join()

    
    

```
![Image](https://dmyhprcifcyj5.cloudfront.net/sites/default/files/cozmo-landingpage-imgs/cozmo-game-mobile.jpg)
![Image](https://tctechcrunch2011.files.wordpress.com/2016/10/cozmo_5_360-png.gif)
![Image](https://dmyhprcifcyj5.cloudfront.net/sites/default/files/cozmo-landingpage-imgs/cozmo-peek-mobile.jpg)
![Image](https://cdn0.vox-cdn.com/thumbor/OVf_RC_ubtstGRqSfyVk5hqoy7s=/1020x0/cdn0.vox-cdn.com/uploads/chorus_asset/file/6770225/anki-cozmo-stock-vpavic-12.0.jpg)
![Image](https://c.slashgear.com/wp-content/uploads/2016/06/anki-cozmo-3.gif)
![Image](https://cdn1.vox-cdn.com/uploads/chorus_asset/file/6947125/cozmo.0.gif)
![Image](http://img.huffingtonpost.com/asset/scalefit_630_noupscale/577137d61500002a0073cbd5.gif)
![Image](https://images-na.ssl-images-amazon.com/images/G/01/vince/boost/detailpages/cozmovideo._SR720,404_.jpg)
[Cozmo Video](https://www.youtube.com/watch?v=cD5xwhZf87U)

### information

[cozmo](https://anki.com/en-us/cozmo?_ga=1.85215710.694554497.1483836170) is a robot that has its own brains. [twilio](https://www.twilio.com/) is a sms/phone service. [python](https://www.python.org/) is an easy text-based language that is very adaptable and widely used. combine the trio and you get this.


Please put a comment in my envelope if you enjoyed my project. I would appreciate it. 
Thanks,
Matthew





