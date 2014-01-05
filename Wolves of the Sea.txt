#!/bin/bash

# Wolves of the Sea
# By Ben Worrall 2013

gameover(){

  clear
  tput cup 3 23
  echo "GAME OVER"
  tput cup 5 6
  echo "Ye be dead, and none shall ever know yer name..."
  echo ""
  echo "Press Enter to continue"
  read enter
  navigation

}

statusbar(){

  # Displays useful stats at the top of the screen

  checkmaps=$(ls $HOME/piracy/plunder | grep "maps")
  if [ ! -z $checkmaps ] ; then
    m=Maps
  fi
  checkdocs=$(ls $HOME/piracy/plunder | grep "documents")
  if [ ! -z $checkdocs ] ; then
    d=Documents
  fi
  tput cup 1 0
  echo "Our Ship: $ship"
  tput cup 1 25
  echo "Points: $totalscore"
  tput cup 1 40
  echo "Kills: $kills"
  tput cup 2 0
  echo "Buried Treasure Found: $treasurefound"
  tput cup 2 26
  echo "Blackmails: $bmail"
  tput cup 2 41
  echo "Crew: $g$s$c"
  tput cup 2 52
 echo "Items: $m / $d"
  tput cup 4 0

}

cleanup(){

  # Removes any files from the game folders

  rm $HOME/piracy/strongbox/* 2> /dev/null
  rm $HOME/piracy/plunder/* 2> /dev/null
  rm $HOME/piracy/enemyhold/* 2> /dev/null

}

firstgame(){

  # Checks to see if the main game folder exists
  # If not, it creates all the files and folders necessary for the game

  if [ ! -d $HOME/piracy ] ; then
    echo "Yarr! I see this be the first time ye be playin'."
    sleep 1
    echo "I just set up the files ye be needin'..."
    mkdir $HOME/piracy
    mkdir $HOME/piracy/crew
    mkdir $HOME/piracy/treasure
    mkdir $HOME/piracy/enemyhold
    mkdir $HOME/piracy/plunder
    mkdir $HOME/piracy/strongbox
    mkdir $HOME/piracy/buried
    touch $HOME/piracy/currentgame.info
    echo "0" > $HOME/piracy/currentgame.info
    touch $HOME/piracy/scores.info
    touch $HOME/piracy/treasure/gold
    touch $HOME/piracy/treasure/silver
    touch $HOME/piracy/treasure/rubies
    touch $HOME/piracy/treasure/emeralds
    touch $HOME/piracy/treasure/silks
    touch $HOME/piracy/treasure/spices
    touch $HOME/piracy/treasure/diamonds
    touch $HOME/piracy/treasure/art
    touch $HOME/piracy/treasure/coins
    touch $HOME/piracy/treasure/sapphires
    touch $HOME/piracy/treasure/cloth
    touch $HOME/piracy/treasure/jewellery
    touch $HOME/piracy/treasure/tapestries
    touch $HOME/piracy/treasure/documents
    touch $HOME/piracy/treasure/maps
    sleep 1
    echo "She be ready, ya scurvy swab! Enjoy the game!"
    sleep 1
  else
    cleanup
  fi

}

assign_values(){

  # Assigns a random value betwen 50-1000 to each piece of treasure in the enemy hold

  for booty in $(ls $HOME/piracy/enemyhold)
  do
    route="$HOME/piracy/enemyhold/$booty"
    value="$((RANDOM%1000+50))"
    echo $value > $route
  done

}

plunder(){

  # Rolls a random number from 1-6
  # You essentially have a 50/50 chance of winning each fight
  # Rolling a 4 or above wins , but each number generates a different fight response

  hold=$HOME/piracy/enemyhold
  roll=$((RANDOM%6+1))
  sleep 1

  case $roll in

    1) echo "The mangy cur kicks you in your baby-maker and runs off to the strongbox with the loot!"
      sleep 1
      echo ""
      echo "The $target can't be plundered!"
      mv $hold/$target $HOME/piracy/strongbox
      echo ""
      echo -n "Press Enter to continue"
    read enter ;;

    2) echo "The filthy sailor easily blocks your sword thrust, knocking it out of your hand."
      sleep 1
      echo "You cry like a girl as he secures the treasure in the strongbox."
      sleep 1
      echo ""
      echo "The $target can't be plundered!"
      mv $hold/$target $HOME/piracy/strongbox
      echo ""
      echo -n "Press Enter to continue"
    read enter ;;

    3) echo "Just as your sword is about to pierce the scrawny wretch, you slip on some bilge water and miss."
      sleep 1
      echo "You curse loudly as he makes off with the loot to the strongbox."
      sleep 1
      echo ""
      echo "The $target can't be plundered!"
      mv $hold/$target $HOME/piracy/strongbox
      echo ""
      echo -n "Press Enter to continue"
    read enter ;;

    4) echo "He blocks a killing blow but it slices off his ear and you grab the loot as he runs away screaming."
      sleep 1
      echo ""
      echo "Yarr! Ye plundered $target worth $(cat $hold/$target)!"
      mv $hold/$target $HOME/piracy/plunder
      echo ""
      echo -n "Press Enter to continue"
    read enter ;;

    5) echo "Ye run the scrawny wretch through and step over his corpse to your booty!"
      sleep 1
      echo ""
      echo "Yarr! Ye plundered $target worth $(cat $hold/$target)!"
      mv $hold/$target $HOME/piracy/plunder
      echo ""
      echo -n "Press Enter to continue"
    read enter ;;

    6) echo "CRITICAL HIT!"
      sleep 1
      echo ""
      echo "The poxy sailor's head flies from his shoulders as you raise the $target above your head, laughing like a mad man!"
      sleep 1
      echo ""
      echo "Yarr! Ye plundered $target worth $(cat $hold/$target)!"
      mv $hold/$target $HOME/piracy/plunder
      echo ""
      echo -n "Press Enter to continue"
    read enter ;;

  esac

}

treasure(){

  echo ""
  treasurefound=$((treasurefound+1))
  rm $HOME/piracy/plunder/* 2> /dev/null
  treasurescore=0
  files=($HOME/piracy/treasure/*)
  cachecontents=$(ls $HOME/piracy/buried | wc -l)
    filetomove="${files[$RANDOM%15+0]}"
    cp $filetomove $HOME/piracy/buried
    for booty in $(ls $HOME/piracy/buried)
    do
      route="$HOME/piracy/buried/$booty"
      value="$((RANDOM%1000+50))"
      echo $value > $route
    done
    rm $HOME/piracy/buried/maps 2> /dev/null
    rm $HOME/piracy/buried/documents 2>/dev/null
    for file in $(ls $HOME/piracy/buried)
    do
      points=$(cat $HOME/piracy/buried/$file)
      amount="$HOME/piracy/plunder/$file"
      echo "We've found $file worth $points!"
      sleep 1
      let treasurescore=$treasurescore+$points
    done
    echo ""
    echo "The treasure be worth $treasurescore!"
    echo ""
	let score=$score+$treasurescore
    echo "With the booty from that enemy $enemyship our total for this trip be $score."
    echo ""
    echo "Press Enter to continue"
    read enter
    echo ""

}

buriedtreasure(){

  # Rolls a dice to see if there is any buried treasure to be found.
  # There's a 25% chance of finding nothing.
  # If there is treasure, a check is made on your ship's max cargo and copies that number of files from /treasure to /buried, assigns them a random value and adds the total to your score for this round.
  # Also begins a tally of buried treasure found for the scoreboard.

  clear
 statusbar
  echo "Cap'n after a long voyage, we've finally arrived at the X on the map!"
  echo ""
  sleep 1
  treasurechance=$((RANDOM%4+1))

  case $treasurechance in

    1) echo "Blast! There be nothin' here!"
      echo "What a wasted journey."
      echo ""
      echo "At least we still got $score from that ship."
      echo ""
      echo "Press Enter to continue"
      read enter
    echo "" ;;

    *) echo "Yarr! Treasure thar be, Cap'n!"
    treasure ;;

  esac

}

mapcheck(){

  # Checks to see if you plundered any maps.
  # There's a 33% chance of that map leading you to some buried treasure.

  checkmaps=$(ls $HOME/piracy/plunder | grep "maps")
  if [ ! -z $checkmaps ] ; then
    mapchance=$((RANDOM%3+1))
    if [ $mapchance -eq 1 ] ; then
      echo "Cap'n! One of the maps we plundered might lead to some treasure!"
      echo ""
      sleep 1
      echo "Shall we seek it out?"
      echo ""
      echo "1. Yarr!      2. Narr..."
      read response

      case $response in

        1) echo ""
          echo "Yarr indeed! Let's set sail!"
          sleep 1
        buriedtreasure ;;

        2) echo ""
          echo "As ye wish Cap'n!"
        sleep 1 ;;

      esac

    fi
  fi

}

nobleattempt(){

  noblechance=$((RANDOM%2+1))

  clear
  statusbar
  case $noblechance in

    1) echo ""
      echo "Yarr! We're in luck! The mangy cur be asleep!"
      sleep 1
      echo ""
      echo "I've slit his poxy throat, Cap'n, and found the key to his treasure chest!"
    treasure ;;

    2) echo ""
      echo "The paranoid wretch was waitin' up fer us!"
      sleep 1
      echo ""
      echo "Press Enter to continue"
      read enter
      roll=$((RANDOM%2+1))
      if [ $roll -eq 1 ] ; then
        echo ""
        echo "You quickly draw your hidden boot knife and throw it through the air, straight into the noble's heart."
        sleep 1
        echo ""
        echo "A quick search reveals the key to his treasure chest!"
        treasure
      else
        clear
        tput cup 3 0
        echo "You turn just in time to see his sword swing at your neck before everything just..."
        sleep 4
        clear
        tput cup 3 23
        echo "... stops."
        sleep 2
        gameover
      fi

  esac

}

jailbreak(){

  escapechance=$((RANDOM%3+1))

  clear
  statusbar
  echo "We're to be hanged at dawn, Cap'n!"
  sleep 1
  echo ""
  echo "We need to escape! Which route do ye choose?"
  echo ""
  echo "1. The door!    2. The window!"
  read decision

  case $decision in

    1) echo ""
      echo "Aye! The door it is!"
      sleep 1
      if [ $escapechance -eq 3 ] ; then
        echo ""
        echo "Curses! The guards were coming down the corridor as we emerged and had no problem tieing us up like pigs!"
        echo ""
        echo "Press Enter to continue"
        read enter
        clear
        tput cup 3 14
        echo "I hear hanging hurts like hell."
        sleep 2
        tput cup 5 8
        echo "Best try jumping and hope yer neck snaps..."
        sleep 3
        gameover
      else
        echo ""
        echo "Thar be no-one here Cap'n!"
        echo ""
        echo "This be the worst jail ever!"
        sleep 1
        echo ""
        echo "We should kill that cowardly noble and take his booty!"
        sleep 1
        echo ""
        echo "What say ye?"
        sleep 1
        echo ""
        echo "1. The bastard dies!    2. It's too risky!"
        read answer

        case $answer in

          1) nobleattempt ;;

          2) echo ""
            echo "Let's wait til nightfall and head back to the ship. We still got $score from that last enemy $enemyship!"
            echo ""
            echo "Press Enter to continue"
          read enter ;;
        esac
    fi ;;

    2) echo "Out the window it is!"
      sleep 1
      if [ $escapechance -eq 3 ] ; then
        echo ""
        echo "Curses! That window was on a cliff-facing wall!"
        echo ""
        echo "Press Enter to continue"
        read enter
        clear
        tput cup 3 10
        echo "We've sure been fallin' a long time now!"
        sleep 2
        tput cup 5 0
        echo "Maybe if we close our eyes it won't hurt when we hit the bottom..."
        sleep 3
        gameover
      else
        echo ""
        echo "It's a good job this window be on the ground floor, Cap'n!"
        echo ""
        echo "We should kill that cowardly noble and take his booty!"
        sleep 1
        echo ""
        echo "What say ye?"
        sleep 1
        echo ""
        echo "1. The bastard dies!    2. It's too risky!"
        read answer

        case $answer in

          1) nobleattempt ;;

          2) echo ""
            echo "Let's wait til nightfall and head back to the ship. We still got $score from that last enemy $enemyship!"
            echo ""
            echo "Press Enter to continue"
          read enter ;;
        esac
      fi

  esac

}

blackmail(){

  clear
  statusbar
  echo "Cap'n! We've arrived at the colony and tracked down the noble!"
  echo ""
  sleep 1
  blackmailchance=$((RANDOM%4+1))
  case $blackmailchance in

    1) echo "Blast! He saw us comin' up the hill and has called the guards!"
      sleep 1
      echo ""
      echo "Thar be too many of 'em Cap'n!"
      sleep 1
      echo ""
      echo "Looks like we be headed for a jail spell."
      sleep 2
      echo ""
      echo "They hang pirates here you know..."
      echo ""
      echo "Press Enter to continue"
      read enter
    jailbreak ;;

    *) echo "Yarr! The miserable swine wet his pants when we showed him the documents and handed over his treasure without a fight!"
      let bmail=$bmail+1
    treasure ;;

  esac

}

docscheck(){

  checkdocs=$(ls $HOME/piracy/plunder | grep "documents")
  if [ ! -z $checkdocs ] ; then
    docschance=$((RANDOM%3+1))
    if [ $docschance -eq 1 ] ; then
      echo "Cap'n! Some of the documents we plundered contain information that might enable us to blackmail a wealthy noble on a nearby island!"
      echo ""
      sleep 1
      echo "Shall we seek him out?"
      echo ""
      echo "1. Yarr!      2. Narr..."
      read response

      case $response in

        1) echo ""
          echo "Yarr indeed! Let's set sail!"
          sleep 1
        blackmail ;;

        2) echo ""
          echo "As ye wish Cap'n!"
        sleep 1 ;;

      esac

    fi
  fi

}

crewdecision(){

  echo ""
  echo "What shall we do with him?"
  echo ""
  echo "1. Welcome aboard, matey!   2. Slaughter him with the rest!"
  read response

  case $response in

    1) echo ""
      echo "Yarr! I'll clear 'im a bunk!"
      decision=1
      echo ""
      echo "Press Enter to continue"
    read enter ;;

    2) echo ""
      echo "As ye command, Cap'n."
      echo ""
      echo "Press Enter to continue"
    read enter ;;

  esac

}

crewcheck(){

  crewroll=$((RANDOM%3+1))
  whichcrew=$((RANDOM%3+1))

  if [ $crewroll -eq 3 ] ; then

    case $whichcrew in

      1) newcrew=Gunner
        if [ $gunner -eq 0 ] ; then
          echo ""
          echo "Cap'n! There's an experienced Gunner among their crew who will join us if we spare his life."
          echo ""
          echo "With him on our crew, our $ship's DAMAGE will increase by +3!"
          crewdecision
          if [ $decision -eq 1 ] ; then
            gunner=1
            g=G
          fi
      fi ;;

      2) newcrew=Shipwright
        if [ $shipwright -eq 0 ] ; then
          echo ""
          echo "Cap'n! There's an experienced Shipwright among their crew who will join us if we spare his life."
          echo ""
          echo "With him on our crew, our $ship's HP will increase by +10!"
          crewdecision
          if [ $decision -eq 1 ] ; then
            shipwright=1
            s=S
          fi
      fi ;;

      3) newcrew=Carpenter
        if [ $carpenter -eq 0 ] ; then
          echo ""
          echo "Cap'n! There's an experienced Carpenter among their crew who will join us if we spare his life."
          echo ""
          echo "With him on our crew, our $ship's CARGO SPACE will increase by +2!"
          crewdecision
          if [ $decision -eq 1 ] ; then
            carpenter=1
            c=C
          fi
      fi ;;

    esac

  fi

}

tally(){

  # Adds up the total value of the treasure you plundered.
  # This is your score for that sortie.
  # Writes this to a file that allows you to add each successive score until you choose to retire (or die!)

  score=0
  for file in $(ls $HOME/piracy/plunder)
  do
    points=$(cat $HOME/piracy/plunder/$file)
    let score=$score+$points
  done
  if [ $score -eq 0 ] ; then
    echo "Yarr! Ye scored... $score!?"
    sleep 1
    echo "Yer not fit to lead a dog!"
    sleep 1
    echo "MUTINY!!"
    sleep 3
    gameover
  fi
  echo "Yarr! Ye scored $score!"
  echo ""
  crewcheck
  docscheck
  mapcheck
  let totalscore=$totalscore+$score
  echo "We can bury this treasure and sail again, or retire now."
  echo ""
  echo "Ye've sunk $kills of the enemy and yer total score be $totalscore."
  echo ""
  echo "What be yer orders Cap'n?"
  echo ""
  echo "1. Sail again!     2. Retire "
  read decision

  case $decision in

    1) echo $totalscore > $HOME/piracy/currentgame.info
      echo "Treasure buried Cap'n! Let's set sail!"
      echo ""
      echo "Press Enter to continue"
      read enter
      cleanup
      m=-
      d=-
    seabattlesetup ;;

    2) echo "Yarr, it's been a pleasure sailin' with ye!"
      sleep 2
      echo "Yer final score be $totalscore."
      echo -n "What be yer name, shipmate? "
      read name
      echo "$name? That's a silly name for a pirate..."
      sleep 2
      echo "$totalscore : $name : $startingship : $kills sunk : $treasurefound buried treasure found : $bmail nobles blackmailed : Crew $g$s$c" >> $HOME/piracy/scores.info
    navigation ;;

  esac

}

boardingsetup(){

  # Sets the parameters for the boarding party
  # Determines how much treasure the enemy needs to be carrying
  # and how much treasure the player ship can cary

  files=($HOME/piracy/treasure/*)
  enemyholdcontents=$(ls $HOME/piracy/enemyhold | wc -l)

  case $enemyship in

    Caravel) cargospace=7 ;;

    Galleon) cargospace=9 ;;

    Frigate) cargospace=3 ;;

    Privateer) cargospace=5 ;;

    Treasureship) cargospace=15 ;;

  esac

  case $ship in

    Caravel) ourcargospace=7 ;;

    Privateer) ourcargospace=5 ;;

    Frigate) ourcargospace=3 ;;

    Galleon) ourcargospace=9 ;;

    Treasureship) ourcargospace=15 ;;

  esac

  if [ $carpenter -eq 1 ] ; then
    let ourcargospace=$ourcargospace+2
  fi
  if [ $enemyholdcontents -lt $cargospace ] ; then
    filetomove="${files[$RANDOM%9+0]}"
    cp $filetomove $HOME/piracy/enemyhold
    boardingsetup
  fi
  assign_values

}

changeship(){

  # Allows the player to change their ship to the one they just plundered.
  # This is the only way to play as the galleon or treasureship.

  echo ""
  echo "What would ye like to do with this enemy $enemyship, Cap'n?"
  sleep 1
  echo ""
  echo "1. Sink her, and all aboard her!"
  echo ""
  echo "2. Set the crew adrift and make this $enemyship our ship!"
  echo ""
  read response

  case $response in

    1) echo "Aye Cap'n! Send 'em to the locker!"
    sleep 2 ;;

    2) echo "Aye! We'll patch her up in no time!"
      ship=$enemyship
      sleep 1
    echo "Our new ship be a $ship" ;;

  esac

}

selection(){

  # Allows the player to select which treasure they would like to try and plunder.

  echo "Our cargo hold contains $ourholdcontents/$ourcargospace"
  echo ""
  echo "What would ye like to plunder?"
  echo ""
  select target in $holdcontents
  do
    if [[ -n $target ]] ; then
      echo ""
      echo "The $target eh?"
      echo ""
      plunder  $target
      statusbar
      select_treasure
    else
      echo ""
      echo "We don't have time fer yer shenanigans, puddin' fingers!"
    fi
  done

}

select_treasure(){

  # Determines if there's any treasure left to plunder and if the player can carry any more.
  # Generates available treasure in a selectable list.

  ourholdcontents=$(ls $HOME/piracy/plunder | wc -l)
  holdcontents=$(ls $HOME/piracy/enemyhold)
  if [ -z "$holdcontents" ] ; then
    clear
    statusbar
    echo "Curses! Thar be nothin' left to plunder!"
    echo ""
    echo "Press Enter to continue"
    read enter
    if [ ! $enemytype -eq $ourtype ] ; then
      changeship
    fi
    echo ""
    echo "Time to make good our escape, and tally the treasure!"
    echo ""
    sleep 2
    tally
    elif [ "$ourholdcontents" -eq "$ourcargospace" ] ; then
    clear
    statusbar
    echo "We can't carry any more booty Cap'n!"
    sleep 2
    if [ ! $enemytype -eq $ourtype ] ; then
      changeship
    fi
    echo ""
    echo "Time to make good our escape, and tally the treasure!"
    echo ""
    sleep 2
    tally
  else
    clear
    statusbar
    echo "Their hold is brimming with booty!"
    echo ""
    echo "A sailor jumps into your path and tries to defend the treasure!"
    echo ""
    sleep 1
    selection
  fi

}

flee(){

  # Rolls a random number from 1-14 to see if the player can escape battle.
  # Makes a check to see if any HP have been lost to return an appropriate dialogue.

  chance=$((RANDOM%14+1))

  case $ship in

    Galleon|Treasureship) echo ""
      echo "Yarr! We've no chance of escape in this wallowing hog!"
      echo
      echo "Press Enter to continue"
      read enter
    shipbattle ;;

    Caravel) if [ $chance -gt 2 ] ; then
        echo ""
        echo "We managed to escape!"
        caravelhp=10
        if [ $ourhp -lt $caravelhp ] ; then
          echo "Let's go make some repairs and choose a different target."
          echo""
          echo "Press Enter to continue"
          read enter
        else
          echo "Let's find a different target."
          echo ""
          echo "Press Enter to continue"
          read enter
        fi
        seabattlesetup
      else
        echo""
        echo "We couldn't break away! Battlestations!"
        echo ""
        echo "Press Enter to continue"
        read enter
        enemyturn
    fi ;;

    Privateer) if [ $chance -gt 4 ] ; then
        echo ""
        echo "We managed to escape!"
        privateerhp=20
        if [ $ourhp -lt $privateerhp ] ; then
          echo "Let's go make some repairs and choose a different target."
          echo ""
          echo "Press Enter to continue"
          read enter
        else
          echo "Let's find a different target."
          echo
          echo "Press Enter to continue"
          read enter
        fi
        seabattlesetup
      else
        echo ""
        echo "We couldn't break away! Battlestations!"
        echo ""
        echo "Press Enter to continue"
        read enter
        enemyturn
    fi ;;

    Frigate) if [ $chance -gt 6 ] ; then
        echo ""
        echo "We managed to escape!"
        frigatehp=30
        if [ $ourhp -lt $frigatehp ] ; then
          echo "Let's go make some repairs and choose a different target."
          echo ""
          echo "Press Enter to continue"
          read enter
        else
          echo "Let's find a different target."
          echo ""
          echo "Press Enter to continue"
          read enter
        fi
        seabattlesetup
      else
        echo ""
        echo "We couldn't break away! Battlestations!"
        echo ""
        echo "Press Enter to continue"
        read enter
        enemyturn
    fi ;;

  esac

}

seabattlesetup(){

  # Sets the parameters for the ship battle.
  # Chooses the type of enemy ship and determines HP.

  cleanup
  enemyroll=$((RANDOM%9+1))

  case $enemyroll in

    1|5) enemyship=Caravel
    enemytype=1 ;;

    2|6) enemyship=Privateer
    enemytype=2 ;;

    3|7) enemyship=Frigate
    enemytype=3 ;;

    4|8) enemyship=Galleon
    enemytype=4 ;;

    9) enemyship=Treasureship
    enemytype=5 ;;

  esac

  case $enemyship in

    Caravel) enemyhp=10 ;;

    Privateer) enemyhp=20 ;;

    Frigate) enemyhp=30 ;;

    Galleon) enemyhp=40 ;;

    Treasureship) enemyhp=60 ;;

  esac

  case $ship in

    Caravel) ourhp=10
    ourtype=1 ;;

    Privateer) ourhp=20
    ourtype=2 ;;

    Frigate) ourhp=30
    ourtype=3 ;;

    Galleon) ourhp=40
    ourtype=4 ;;

    Treasureship) ourhp=60
    ourtype=5 ;;

  esac

  if [ $shipwright -eq 1 ] ; then
    let ourhp=$ourhp+10
  fi
  clear
  statusbar

  case $enemyship in

    Treasureship) echo "Hell's bells Cap'n! She's a $enemyship, and she's laden with booty!" ;;

    *)   echo "Yarr! She's a $enemyship!" ;;

  esac

  tput cup 6 0
  echo "Our HP: $ourhp points"
  tput cup 6 20
  echo "Enemy HP: $enemyhp points"
  tput cup 8 0
  echo "What be yer orders, Cap'n?"
  echo -n "1. Engage the enemy!!    2. Run awaaaaay! "
  read orders

  case $orders in

    1) shipbattle ;;

    2) flee ;;

    *) echo ""
      echo "Ye be speakin' gibberish, Captain!"
      echo "We'll try and outrun her."
      sleep 1
    flee ;;

  esac

}

playerturn(){

  # Rolls two random numbers from 1-3.
  # If they match, the player misses, otherwise it's a hit.
  # Essentially a 2:3 chance of a hit
  # Subtracts damage done from HP

  number=$((RANDOM%3+1))
  hitormiss=$((RANDOM%3+1))
  if [ ! $number -eq $hitormiss ] ; then
    echo ""
    echo "A direct hit for $damage points!"
    echo ""
    echo -n "Press Enter to continue"
    read enter
    let enemyhp=$enemyhp-$damage
    if [ $enemyhp -lt 1 ] ; then
      echo ""
      echo "Yarr! She's dead in the water!"
      echo "Let's board the sow and claim what's ours!"
      echo ""
      echo "Pres Enter to continue."
      read enter
      kills=$((kills+1))
      boardingsetup
      select_treasure
    fi
  else
    echo ""
    echo "Blast! We missed!"
    echo ""
    echo -n "Press Enter to continue"
    read enter
  fi

}

enemyturn(){

  # Rolls two random numbers from 1-3.
  # If they match, the enemy misses, otherwise it's a hit.
  # Essentially a 2:3 chance of a hit
  # Subtracts damage done from HP

  echo ""
  echo "The enemy is about to fire!"
  number=$((RANDOM%3+1))
  hitormiss=$((RANDOM%3+1))
  if [ ! $number -eq $hitormiss ] ; then
    echo ""
    echo "They hit us! We've been damaged for $enemydamage points!"
    echo ""
    echo -n "Press Enter to continue"
    read enter
    let ourhp=$ourhp-$enemydamage
    if [ $ourhp -lt 1 ] ; then
      clear
      tput cup 3 15
      echo "Abandon ship! We're done for!"
      tput cup 5 12
      echo "What a sorry excuse for a captain!"
      tput cup 7 4
      echo "Ye've killed us aaaaaaalllll!! *Gaaargh gargle gargle*"
      echo""
      echo "Press Enter to continue"
      read enter
      gameover
    fi
    shipbattle
  else
    echo ""
    echo "Yarr! They missed us!!"
    echo ""
    echo -n "Press Enter to continue"
    read enter
    shipbattle
  fi

}

shipbattle(){

  # Displays the details of the ship battle

  case $enemyship in

    Caravel) enemydamage=$((RANDOM%3+1)) ;;

    Privateer) enemydamage=$((RANDOM%6+4)) ;;

    Frigate) enemydamage=$((RANDOM%7+5)) ;;

    Galleon) enemydamage=$((RANDOM%4+2)) ;;

    Treasureship) enemydamage=$((RANDOM%5+3)) ;;

  esac

  case $ship in

    Caravel) damage=$((RANDOM%3+1)) ;;

    Privateer) damage=$((RANDOM%6+4)) ;;

    Frigate) damage=$((RANDOM%7+5)) ;;

    Galleon) damage=$((RANDOM%4+2)) ;;

    Treasureship) damage=$((RANDOM%5+3)) ;;

  esac

  if [ $gunner -eq 1 ] ; then
    let damage=$damage+3
  fi
  clear
  statusbar
  echo "Ye be fightin' a $enemyship!"
  tput cup 6 0
  echo "Our HP: $ourhp points"
  tput cup 6 20
  echo "Enemy HP: $enemyhp points"
  tput cup 8 0
  echo "What be yer orders, Cap'n?"
  echo -n "1. FIIIIRRE!!    2. Run awaaaaay! "
  read attackorders

  case $attackorders in

    1) echo ""
    echo "KABOOM!!" ;;

    2) flee ;;

    *) echo "Ye be speakin' gibberish Captain!"
    shipbattle ;;

  esac

  playerturn
  enemyturn

}

shipselect(){

  # Displays a menu to allow the player to select a ship to play.
  # Each option displays stats for the selected ship.

  clear
  tput cup 3 23
  echo "Choose Yer Vessel!"
  tput cup 4 12
  echo "************************************"
  tput cup 7 20
  echo "1. Caravel"
  tput cup 9 20
  echo "2. Privateer"
  tput cup 11 20
  echo "3. Frigate"
  tput cup 13 20
  echo "4. Return to menu"
  read pickship

  case $pickship in

    1)echo ""
      echo "The caravel is fleet in the water and hard to hit. She has fewer guns and lower HP, but plenty of room for treasure."
      echo ""
      tput cup 19 20
      echo "HP : 10"
      tput cup 20 20
      echo "Damage : 1-3 (critical 5)"
      tput cup 21 20
      echo "Cargo Capacity : 7"
      tput cup 22 20
      echo "Escape Chance : 75%"
      tput cup 23 20
      echo "Starting Difficulty : HARD"
      echo ""
      echo -n "Do you want to take this ship? "
      read input
      answer=$(echo $input | cut -c1)

      case $answer in

        y|Y) ship=Caravel
          startingship=Caravel
          clear
          tput cup 3 30
          echo "SHIP AHOY!"
          tput cup 5 12
          echo "Let's take the fat bellied whore down to Davy Jones!"
          echo ""
          echo "Press Enter to continue"
          read enter
          clear
          statusbar
        seabattlesetup ;;

        n|N) shipselect ;;

        *) echo "I'll take that as a No, ye scallywag!"
          sleep 1
        shipselect ;;

    esac ;;

    2)echo ""
      echo "The privateer is a pirate ship through and through, with good guns and good speed, and able to hold a decent amount of booty!"
      echo ""
      tput cup 19 20
      echo "HP : 20"
      tput cup 20 20
      echo "Damage : 4-6 (critical 9)"
      tput cup 21 20
      echo "Cargo Capacity : 5"
      tput cup 22 20
      echo "Escape Chance : 50%"
      tput cup 23 20
      echo "Starting Difficulty : MEDIUM"
      echo ""
      echo -n "Do you want to take this ship? "
      read input
      answer=$(echo $input | cut -c1)

      case $answer in

        y|Y) ship=Privateer
          startingship=Privateer
          clear
          tput cup 3 30
          echo "SHIP AHOY!"
          tput cup 5 12
          echo "Let's take the fat bellied whore down to Davy Jones!"
          echo ""
          echo "Press Enter to continue"
          read enter
          clear
          statusbar
        seabattlesetup ;;

        n|N) shipselect ;;

        *) echo "I'll take that as a No, ye scallywag!"
          sleep 1
        shipselect ;;

    esac ;;

    3)echo ""
      echo "The frigate is a warship with frightening strength and guns, but she's slower in the water and liable to take a beatin', and her hold is full of ammo so there's less room for treasure."
      echo ""
      tput cup 19 20
      echo "HP : 30"
      tput cup 20 20
      echo "Damage : 5-7 (critical 11)"
      tput cup 21 20
      echo "Cargo Capacity : 3"
      tput cup 22 20
      echo "Escape Chance : 25%"
      tput cup 23 20
      echo "Starting Difficulty : EASY"
      echo ""
      echo -n "Do you want to take this ship? "
      read input
      answer=$(echo $input | cut -c1)

      case $answer in

        y|Y) ship=Frigate
          startingship=Frigate
          clear
          tput cup 3 30
          echo "SHIP AHOY!"
          tput cup 5 12
          echo "Let's take the fat bellied whore down to Davy Jones!"
          echo ""
          echo "Press Enter to continue"
          read enter
          clear
          statusbar
        seabattlesetup ;;

        n|N) shipselect ;;

        *) echo "I'll take that as a No, ye scallywag!"
          sleep 1
        shipselect ;;

    esac ;;

    4) navigation ;;

    *) shipselect ;;

  esac

}

navigation(){

  # The main game menu

  clear
  tput cup 3 10
  echo -e "Welcome to Wolves of the Sea, ya wastrel!"
  tput cup 4 12
  echo "*************************************"
  tput cup 7 20
  echo "1. Start Game"
  tput cup 9 20
  echo "2. View Scores"
  tput cup 11 20
  echo "3. Exit"
  read option

  case $option in

    1) cleanup
      rm $HOME/piracy/inventory/* 2> /dev/null
      echo "0" > $HOME/piracy/currentgame.info
      totalscore=0
      kills=0
      treasurefound=0
      bmail=0
      gunner=0
      carpenter=0
      shipwright=0
      g=-
      c=-
      s=-
      m=-
      d=-
    shipselect ;;

    2) clear
      tput cup 3 23
      echo "HIGH SCORES"
      echo ""
      sort -r -n $HOME/piracy/scores.info
      echo ""
      echo "Press enter to return to menu"
      read enter
    navigation ;;

    3) echo "Fair winds and safe harbours to ye."
      sleep 2
      clear
    exit ;;

  esac

}

clear
firstgame
navigation

	