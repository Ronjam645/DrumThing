# DrumThing
Digital drum machine
#import "gridView.h"
#import <stdio.h>
#import <string.h>
#import <AudioToolbox/AudioToolbox.h>

@implementation gridView

- (id)initWithFrame:(NSRect)frameRect
{
	[super initWithFrame:frameRect];
	return self;
}

- (void)dealloc
{
    [mTimer release];
    [super dealloc];
}

- (BOOL)acceptsFirstResponder
{
return YES;
}

- (BOOL)isOpaque
{
    return YES;
}

- (void) awakeFromNib
{

TEMPO=160; BARS=2; SUB=2; GRID=1; BEATS=4; DEMO=1;  PRE=4; POS=1; colTOG=0; undoTOG=0; randTOG=0; dataTOG=0;
oldD1=DR1; DR1=51; oldD2=DR2; DR2=37; oldD3=DR3; DR3=44;  oldD4=DR4; DR4=68;  oldD5=DR5;   DR5=67; 
oldD6=DR6; DR6=78; oldD7=DR7; DR7=79; oldD8=DR8; DR8=47;  oldD9=DR9; DR9=63;  oldD10=DR10; DR10=36;

oldV1=DV1; oldV2=DV2; oldV3=DV3; oldV4=DV4; oldV5=DV5; oldV6=DV6; oldV7=DV7; oldV8=DV8; oldV9=DV9; oldV10=DV10;  
DV1=DV2=DV5=DV6=DV7=DV8=DV9=DV10=90; DV3=120; DV4=60;  DRC=51;

[self setDefaults:self]; 
}

- (IBAction)quit:(id)sender;
{
    if(dataTOG==0)exit(0);
    int r = NSRunAlertPanel(@"YOU HAVE UNSAVED CHANGES TO YOUR PROJECT!",@"WANT TO SAVE YOUR PROJECT FIRST?",@"SAVE",@"QUIT",@"CANCEL"); //printf("r=%i ",r);
    if(r==1)[self saveFile:self];
    if(r >1)return;
    if(r==0)exit(0);
}

- (void)drawRect:(NSRect)rect
{

if(colTOG)[[NSColor blackColor] set];
else      [[NSColor whiteColor] set]; 

NSRectFill([self bounds]);  
    
NSMutableDictionary * attribs = [NSMutableDictionary dictionary];
              
int g; int center = 404;

while(SUB*BARS*BEATS > 54)BARS--;          //printf("\nSUB*BARS*BEATS=%i  ",SUB*BARS*BEATS);

left = center - ((SUB*BARS*BEATS)*14)/2;  // 14 = box size
int right = left + 14*(BARS*SUB*BEATS); 
int top = 318; int bottom = 14; 
 
 if(colTOG)[[NSColor whiteColor] set];
 else      [[NSColor grayColor] set];
   
 if(GRID){

    for(g=0; g < 20; g++)  // horizontal lines
    {
    if(g==0 || g==19)[NSBezierPath  setDefaultLineWidth:2.2];
    else             [NSBezierPath  setDefaultLineWidth:0.8];
    NSPoint leftPoint  = NSMakePoint( left-1, top - g*16 );
    NSPoint rightPoint = NSMakePoint( right+1,top - g*16 );
    [NSBezierPath strokeLineFromPoint:leftPoint toPoint:rightPoint];
    
    if(colTOG)[attribs setObject:[NSColor cyanColor]forKey:NSForegroundColorAttributeName];
     else     [attribs setObject:[NSColor blackColor]forKey:NSForegroundColorAttributeName];
	
    NSString *DN = [NSString stringWithFormat:@"Dr%2d", (g/2)+1];
    if(g % 2 == 0)[DN drawAtPoint:NSMakePoint(left-30, (top - g*16) - 15)withAttributes: attribs];
    [attribs setObject:[NSFont fontWithName:@"Lucida Grande" size:10]forKey:NSFontAttributeName];
                                              //printf("\nL=%i  R=%i top=%i  bot=%i  ", left, right, top, bottom); 
    }
    
    LNG = BEATS*SUB*(BARS);
        
    for(g=0; g < LNG; g++) // vertical lines
    {
    
    [NSBezierPath  setDefaultLineWidth:0.8]; 
    [[NSColor grayColor] set];
    
    if(g % (SUB*BEATS) == 0 ){
       NSString *BN = [NSString stringWithFormat:@"%d", g/(SUB*BEATS)+1];  // bar numbers
       [attribs setObject:[NSFont fontWithName:@"Lucida Grande" size:8]forKey:NSFontAttributeName];
       [BN drawAtPoint:NSMakePoint(left-1 + g*14,top+4)withAttributes: attribs];
      [NSBezierPath  setDefaultLineWidth:2.2];  
      
      if(colTOG)[[NSColor whiteColor] set];
      else      [[NSColor blackColor] set];
      
       }
       
      if(g && g % SUB == 0 && g % (BEATS*SUB) != 0){
     // if(colTOG)[[NSColor yellowColor] set];
     // else      
      [[NSColor redColor] set];
       }  
    
      NSPoint topPointLeft    = NSMakePoint(left + g*14,top-1);
      NSPoint bottomPointLeft = NSMakePoint(left + g*14,bottom+1);
      [NSBezierPath strokeLineFromPoint:topPointLeft toPoint:bottomPointLeft];
    
     [attribs setObject:[NSFont fontWithName:@"Lucida Grande" size:25]forKey:NSFontAttributeName];
     
     if(colTOG)[attribs setObject:[NSColor cyanColor]forKey:NSForegroundColorAttributeName];
     else      [attribs setObject:[NSColor blackColor]forKey:NSForegroundColorAttributeName];
     
    if(DR1BUF[g] && DV1) [@"•" drawAtPoint:NSMakePoint(left + g*14,top- 21)withAttributes: attribs];
    if(DR2BUF[g] && DV2) [@"•" drawAtPoint:NSMakePoint(left + g*14,top- 53)withAttributes: attribs];
    if(DR3BUF[g] && DV3) [@"•" drawAtPoint:NSMakePoint(left + g*14,top- 85)withAttributes: attribs];
    if(DR4BUF[g] && DV4) [@"•" drawAtPoint:NSMakePoint(left + g*14,top-117)withAttributes: attribs];
    if(DR5BUF[g] && DV5) [@"•" drawAtPoint:NSMakePoint(left + g*14,top-149)withAttributes: attribs];
    if(DR6BUF[g] && DV6) [@"•" drawAtPoint:NSMakePoint(left + g*14,top-181)withAttributes: attribs];
    if(DR7BUF[g] && DV7) [@"•" drawAtPoint:NSMakePoint(left + g*14,top-213)withAttributes: attribs];
    if(DR8BUF[g] && DV8) [@"•" drawAtPoint:NSMakePoint(left + g*14,top-245)withAttributes: attribs];
    if(DR9BUF[g] && DV9) [@"•" drawAtPoint:NSMakePoint(left + g*14,top-277)withAttributes: attribs];
    if(DR10BUF[g] && DV10)[@"•" drawAtPoint:NSMakePoint(left + g*14,top-309)withAttributes: attribs];
    
     if(velTOG){
     [attribs setObject:[NSFont fontWithName:@"Lucida Grande" size:7]forKey:NSFontAttributeName];
     
     NSString *V1 = [NSString stringWithFormat:@"%3d", DR1VOL[g]]; 
     if(DR1BUF[g] && DV1){ [ V1 drawAtPoint:NSMakePoint(left + g*14,top-28)withAttributes: attribs]; }
     
     NSString *V2 = [NSString stringWithFormat:@"%3d", DR2VOL[g]];
     if(DR2BUF[g] && DV2){ [ V2 drawAtPoint:NSMakePoint(left + g*14,top-60)withAttributes: attribs]; } 
     
     NSString *V3 = [NSString stringWithFormat:@"%3d", DR3VOL[g]];
     if(DR3BUF[g] && DV3){ [ V3 drawAtPoint:NSMakePoint(left + g*14,top-92)withAttributes: attribs]; } 
     
     NSString *V4 = [NSString stringWithFormat:@"%3d", DR4VOL[g]];
     if(DR4BUF[g] && DV4){ [ V4 drawAtPoint:NSMakePoint(left + g*14,top-124)withAttributes: attribs]; } 
     
     NSString *V5 = [NSString stringWithFormat:@"%3d", DR5VOL[g]];
     if(DR5BUF[g] && DV5){ [ V5 drawAtPoint:NSMakePoint(left + g*14,top-156)withAttributes: attribs]; }
          
     NSString *V6 = [NSString stringWithFormat:@"%3d", DR6VOL[g]];
     if(DR6BUF[g] && DV6){ [ V6 drawAtPoint:NSMakePoint(left + g*14,top-188)withAttributes: attribs]; }
          
     NSString *V7 = [NSString stringWithFormat:@"%3d", DR7VOL[g]];
     if(DR7BUF[g] && DV7){ [ V7 drawAtPoint:NSMakePoint(left + g*14,top-220)withAttributes: attribs]; }
          
     NSString *V8 = [NSString stringWithFormat:@"%3d", DR8VOL[g]];
     if(DR8BUF[g] && DV8){ [ V8 drawAtPoint:NSMakePoint(left + g*14,top-252)withAttributes: attribs]; }
          
     NSString *V9 = [NSString stringWithFormat:@"%3d", DR9VOL[g]];
     if(DR9BUF[g] && DV9){ [ V9 drawAtPoint:NSMakePoint(left + g*14,top-284)withAttributes: attribs]; }
     
      NSString *V10 = [NSString stringWithFormat:@"%3d", DR10VOL[g]];
     if(DR10BUF[g] && DV10){ [ V10 drawAtPoint:NSMakePoint(left + g*14,top-316)withAttributes: attribs]; }

     [self setNeedsDisplay:YES];
    }
    
   } 
    
    [NSBezierPath  setDefaultLineWidth:2.2]; 
    
     if(colTOG)[[NSColor whiteColor] set];
     else      [[NSColor blackColor] set];
     
    NSPoint topPointLeft    = NSMakePoint(left + 14*g+1,top+1);
    NSPoint bottomPointLeft = NSMakePoint(left + 14*g+1,bottom-1);
    [NSBezierPath strokeLineFromPoint:topPointLeft toPoint:bottomPointLeft];
        
    }

}

// MOUSE EVENTS  ====================================================================================

- (void)mouseDown:(NSEvent *)event;
{
G=0; int x,y;

NSPoint eventLocation = [event locationInWindow];
pt = [self convertPoint:eventLocation fromView:nil];
x=pt.x; y=pt.y;  //[ X setFloatValue:x]; [ Y setFloatValue:y]; 
//printf("\ny=%i x=%i", y,x);

 G = (x - left) / 14;   // [ gridField setFloatValue:G];

if(y < 318 && y > 300){ if(DR1BUF[G])DR1BUF[G]=0;   else DR1BUF[G]=DR1;   if(DR1BUF[G])  DEMO=1;  DR1VOL[G] = oldV1 = DV1; }
if(y < 286 && y > 268){ if(DR2BUF[G])DR2BUF[G]=0;   else DR2BUF[G]=DR2;   if(DR2BUF[G])  DEMO=2;  DR2VOL[G] = oldV2 = DV2; }
if(y < 252 && y > 238){ if(DR3BUF[G])DR3BUF[G]=0;   else DR3BUF[G]=DR3;   if(DR3BUF[G])  DEMO=3;  DR3VOL[G] = oldV3 = DV3; }
if(y < 222 && y > 204){ if(DR4BUF[G])DR4BUF[G]=0;   else DR4BUF[G]=DR4;   if(DR4BUF[G])  DEMO=4;  DR4VOL[G] = oldV4 = DV4; }
if(y < 188 && y > 174){ if(DR5BUF[G])DR5BUF[G]=0;   else DR5BUF[G]=DR5;   if(DR5BUF[G])  DEMO=5;  DR5VOL[G] = oldV5 = DV5; }
if(y < 156 && y > 140){ if(DR6BUF[G])DR6BUF[G]=0;   else DR6BUF[G]=DR6;   if(DR6BUF[G])  DEMO=6;  DR6VOL[G] = oldV6 = DV6; }
if(y < 126 && y > 110){ if(DR7BUF[G])DR7BUF[G]=0;   else DR7BUF[G]=DR7;   if(DR7BUF[G])  DEMO=7;  DR7VOL[G] = oldV7 = DV7; }
if(y <  94 && y >  78){ if(DR8BUF[G])DR8BUF[G]=0;   else DR8BUF[G]=DR8;   if(DR8BUF[G])  DEMO=8;  DR8VOL[G] = oldV8 = DV8; }
if(y <  62 && y >  46){ if(DR9BUF[G])DR9BUF[G]=0;   else DR9BUF[G]=DR9;   if(DR9BUF[G])  DEMO=9;  DR9VOL[G] = oldV9 = DV9; }
if(y <  30 && y >  14){ if(DR10BUF[G])DR10BUF[G]=0; else DR10BUF[G]=DR10; if(DR10BUF[G]) DEMO=10; DR10VOL[G] = oldV10 = DV10; }

[self setNeedsDisplay:YES];

if(DEMO){  doDEMO(); dataTOG=1; }


}  // END MOUSE EVENTS  ====================================================================================

// KEYDOWN=============================================================================================

- (void)keyDown:(NSEvent *)event;
{

 NSArray * names = [DRnames componentsSeparatedByString:@","];

key = [[event characters] characterAtIndex:0]; // printf("\nkey= %i ", key);

if(key == 49)DEMO=1;  // 1
if(key == 50)DEMO=2;  // 2
if(key == 51)DEMO=3;  // 3
if(key == 52)DEMO=4;  // 4
if(key == 53)DEMO=5;  // 5
if(key == 54)DEMO=6;  // 6
if(key == 55)DEMO=7;  // 7
if(key == 56)DEMO=8;  // 8
if(key == 57)DEMO=9;  // 9
if(key == 48)DEMO=10;  // 0 printf("\nDEMO= %i ", DEMO);

if(key > 47 && key < 58){  doDEMO(); }

if(key == 105)if(INF)INF=0; else INF=1; // i key
if(INF)[inf setStringValue:@"INFINITE RANDOM"]; else [inf setStringValue:@" "]; 

if(key == 63232 || key == 63233 || key == 63234 || key == 63235) // set drums with arrow keys
{

if(DEMO== 1){ DRC=DR1-35; DRV=DV1; }
if(DEMO== 2){ DRC=DR2-35; DRV=DV2; }
if(DEMO== 3){ DRC=DR3-35; DRV=DV3; }
if(DEMO== 4){ DRC=DR4-35; DRV=DV4; }
if(DEMO== 5){ DRC=DR5-35; DRV=DV5; }
if(DEMO== 6){ DRC=DR6-35; DRV=DV6; }
if(DEMO== 7){ DRC=DR7-35; DRV=DV7; }
if(DEMO== 8){ DRC=DR8-35; DRV=DV8; }
if(DEMO== 9){ DRC=DR9-35; DRV=DV9; }
if(DEMO==10){ DRC=DR10-35; DRV=DV10; }

if(key == 63235)if(DRC++ > 48)DRC=0;
if(key == 63234)if(DRC-- <  1)DRC=49; //printf("\nDRC= %i ", DRC);

if( (key == 63234 || key == 63235) && KIT < 1000)
{
    if(DEMO== 1){ DR1=DRC+35; [self displayNotes]; }
    if(DEMO== 2){ DR2=DRC+35; [self displayNotes]; }
    if(DEMO== 3){ DR3=DRC+35; [self displayNotes]; }
    if(DEMO== 4){ DR4=DRC+35; [self displayNotes]; }
    if(DEMO== 5){ DR5=DRC+35; [self displayNotes]; }
    if(DEMO== 6){ DR6=DRC+35; [self displayNotes]; }
    if(DEMO== 7){ DR7=DRC+35; [self displayNotes]; }
    if(DEMO== 8){ DR8=DRC+35; [self displayNotes]; }
    if(DEMO== 9){ DR9=DRC+35; [self displayNotes]; }
    if(DEMO== 10){ DR10=DRC+35; [self displayNotes]; }
 }
 
 if( (key == 63234 || key == 63235) && KIT > 1000)
 {
if(DEMO== 1){ DR1=DRC+35; [dr1  setIntValue:DRC]; [ drum1Field  setStringValue:[names objectAtIndex:DRC] ]; }
if(DEMO== 2){ DR2=DRC+35; [dr2  setIntValue:DRC]; [ drum2Field  setStringValue:[names objectAtIndex:DRC] ]; }
if(DEMO== 3){ DR3=DRC+35; [dr3  setIntValue:DRC]; [ drum3Field  setStringValue:[names objectAtIndex:DRC] ]; }
if(DEMO== 4){ DR4=DRC+35; [dr4  setIntValue:DRC]; [ drum4Field  setStringValue:[names objectAtIndex:DRC] ]; }
if(DEMO== 5){ DR5=DRC+35; [dr5  setIntValue:DRC]; [ drum5Field  setStringValue:[names objectAtIndex:DRC] ]; }
if(DEMO== 6){ DR6=DRC+35; [dr6  setIntValue:DRC]; [ drum6Field  setStringValue:[names objectAtIndex:DRC] ]; }
if(DEMO== 7){ DR7=DRC+35; [dr7  setIntValue:DRC]; [ drum7Field  setStringValue:[names objectAtIndex:DRC] ]; }
if(DEMO== 8){ DR8=DRC+35; [dr8  setIntValue:DRC]; [ drum8Field  setStringValue:[names objectAtIndex:DRC] ]; }
if(DEMO== 9){ DR9=DRC+35; [dr9  setIntValue:DRC]; [ drum9Field  setStringValue:[names objectAtIndex:DRC] ]; }
if(DEMO==10){ DR10=DRC+35;[dr10  setIntValue:DRC]; [ drum10Field  setStringValue:[names objectAtIndex:DRC] ]; }
 }

if( key == 63232 )if(DRV++ > 126)DRV=0;
if( key == 63233 )if(DRV-- <   1)DRV=127; // printf("  DRV= %i ", DRV);

if(key == 63232 || key == 63233)
{
if(DEMO== 1){ DV1=DRV; [self displayVolume]; }
if(DEMO== 2){ DV2=DRV; [self displayVolume]; }
if(DEMO== 3){ DV3=DRV; [self displayVolume]; }
if(DEMO== 4){ DV4=DRV; [self displayVolume]; }
if(DEMO== 5){ DV5=DRV; [self displayVolume]; }
if(DEMO== 6){ DV6=DRV; [self displayVolume]; }
if(DEMO== 7){ DV7=DRV; [self displayVolume]; }
if(DEMO== 8){ DV8=DRV; [self displayVolume]; }
if(DEMO== 9){ DV9=DRV; [self displayVolume]; }
if(DEMO==10){ DV10=DRV; [self displayVolume]; }
}

}

// toggle drum ON and OFF with opt key

if(key== 161){ if(DV1==0)DV1=oldV1; else { oldV1=DV1;  DV1=0; } [self displayVolume]; } //printf("  oldV1= %i ", oldV1);
if(key==8482){ if(DV2==0)DV2=oldV2; else { oldV2=DV2;  DV2=0; } [self displayVolume]; }
if(key== 163){ if(DV3==0)DV3=oldV3; else { oldV3=DV3;  DV3=0; } [self displayVolume]; }
if(key== 162){ if(DV4==0)DV4=oldV4; else { oldV4=DV4;  DV4=0; } [self displayVolume]; }
if(key==8734){ if(DV5==0)DV5=oldV5; else { oldV5=DV5;  DV5=0; } [self displayVolume]; }
if(key== 167){ if(DV6==0)DV6=oldV6; else { oldV6=DV6;  DV6=0; } [self displayVolume]; }
if(key== 182){ if(DV7==0)DV7=oldV7; else { oldV7=DV7;  DV7=0; } [self displayVolume]; }
if(key==8226){ if(DV8==0)DV8=oldV8; else { oldV8=DV8;  DV8=0; } [self displayVolume]; }
if(key== 170){ if(DV9==0)DV9=oldV9; else { oldV9=DV1;  DV9=0; } [self displayVolume]; }
if(key== 186){ if(DV10==0)DV10=oldV10; else { oldV10=DV10;  DV10=0; } [self displayVolume]; }

 unsigned int flag = [[NSApp currentEvent] modifierFlags];  //printf("\nflag= %i ", flag);
 
 if ( (flag & NSAlternateKeyMask) || (flag & NSControlKeyMask) || (flag & NSCommandKeyMask) )

 if(key == 223){ saveSET=1;     [self saveFile:self]; } // save/load drum sets
 if(key == 248){ saveSET=1;     [self openFile:self]; }
 if(key ==  19){ exportMIDI=1;  [self saveFile:self]; }
 
 if(key==114){ randTOG=3; randomize(); [undo setTitle:@"Revert"]; } // R key, randomize
 if(key== 97)
 { randTOG=5; randomize(); if(KIT > 1000)[self displayDrums];  
 else [self displayNotes]; [self displayVolume];[undo setTitle:@"Revert"]; } // A key, set all random
 
 if(key == 174){ randTOG=1; randomize(); [self displayVolume]; }  // random vol
 if(key ==  18){ randTOG=4; randomize(); [self displayVolume]; }
 if(key == 100){ randTOG=2; randomize(); if(KIT > 1000)[self displayDrums];  else [self displayNotes];  } // D key
 
 if(randTOG)dataTOG=1;
 
 if(key == 99)if(colTOG)colTOG=0; else colTOG=1; // printf("\ncolTOG= %i ", colTOG);
 
 //START & STOP PLAY ******************************************
 
if(key ==  32){ [self playGroove:self]; } //initQT();
  
 if(key == 112){  [self playGroove:self]; } // P key,  pause
 
  //                  ******************************************
  
if(key==91)if(BARS-- < 2)BARS=1;
if(key==93)if(BARS++ > 6)BARS=6;
  
 if(key==118){ if(velTOG)velTOG=0; else velTOG=1; }
  
if(key==127){ [undo setTitle:@"Undo"];   clearBufs(); }
if(key==117){ [undo setTitle:@"Revert"]; [self undoClear:self]; }
 
  if(key==44 || key==46){
  if(key==44)if(TEMPO-- < 41)TEMPO=240;
  if(key==46)if(TEMPO++ >239)TEMPO=40;
  [ tempoField  setIntValue:TEMPO]; [ tempoSlider  setIntValue:TEMPO];
  }
    
  if(key==47 || key==63)
  NSBeginInformationalAlertSheet(@"DrumThing Help", nil, nil, nil, [NSApp mainWindow], self, nil, nil, nil, helptext);
 
 [self setNeedsDisplay:YES]; 
 
 } // end of keyDown -----------------------------------------
 

#pragma mark TIMER

- (IBAction)playGroove:(id)sender
{

if ([mTimer isValid]) 
{
                                           
        [mTimer invalidate]; TOG=1;
       // CloseComponent(na); NADisposeNoteChannel(na,nc); // Stop the current timer, clear QT stuff
        
        if(key == 112) { [play setTitle:@"paused"]; LOOP = LOOP; }
                  else { [play setTitle:@"PLAY"];   LOOP = 0; [ count setStringValue:@"0:0"]; } 
                  
	}
	else
	{
        //initQT(); 
                   
		[mTimer autorelease];  		  // Make a new timer
		       mTimer = [[NSTimer
				scheduledTimerWithTimeInterval:0.000
				target:self
				selector:@selector(respondToTimer:)
				userInfo:nil
				repeats:YES
			] retain];
             
	[play setTitle:@"STOP"]; DEMO=TOG=0;
                
	}
}

int initQT()
{

er = 0; na = 0; nc = 0; 

    na = OpenDefaultComponent(kNoteAllocatorType, 0);
    
    if(na == NULL) printf("open Comp error ");  
     
    nr.info.flags = 0;
 //   nr.info.polyphony = EndianS16_NtoB(10);
 //   nr.info.typicalPolyphony = EndianU32_NtoB(0x00010000);
    
    er = NAStuffToneDescription(na, KIT, &nr.tone); er = NANewNoteChannel(na, &nr, &nc);

    
if(er){
NSRunAlertPanel( @"WHAT THE ****!!!",@"THERE SEEMS TO BE A QUICKTIME PROBLEM\nCheck to see if you have QT installed",@"THAT SUCKS!",nil,nil);
if(nc) NADisposeNoteChannel(na,nc); if(na) CloseComponent(na);
[mTimer invalidate]; 
}

}

- (IBAction)setDefaults:(id)sender;
{
oldD1=DR1; DR1=51; oldD2=DR2; DR2=37; oldD3=DR3; DR3=44;  oldD4=DR4; DR4=68;  oldD5=DR5;   DR5=67; 
oldD6=DR6; DR6=78; oldD7=DR7; DR7=79; oldD8=DR8; DR8=47;  oldD9=DR9; DR9=63;  oldD10=DR10; DR10=36;

oldV1=DV1; oldV2=DV2; oldV3=DV3; oldV4=DV4; oldV5=DV5; oldV6=DV6; oldV7=DV7; oldV8=DV8; oldV9=DV9; oldV10=DV10;  
DV1=DV2=DV5=DV6=DV7=DV8=DV9=DV10=90; DV3=120; DV4=60;  DRC=51; 

oldKIT = KIT; KIT=16417; initQT();

initVOL(); setVOL(); [self displayVolume]; 
if(KIT > 1000)[self displayDrums];  else [self displayNotes]; 
}

- (IBAction)setPrev:(id)sender;
{
if(oldV1 < 1 || oldD1 < 1)[self setDefaults:self];

DR1=oldD1; DR2=oldD2; DR3=oldD3; DR4=oldD4; DR5=oldD5; DR6=oldD6; DR7=oldD7; DR8=oldD8; DR9=oldD9; DR10=oldD10;
DV1=oldV1; DV2=oldV2; DV3=oldV3; DV4=oldV4; DV5=oldV5; DV6=oldV6; DV7=oldV7; DV8=oldV8; DV9=oldV9; DV10=oldV10; 

KIT = oldKIT;  initQT();    //  printf("KIT=%i  ",KIT);

initVOL(); setVOL(); [self displayVolume]; 
if(KIT > 1000)[self displayDrums];  else [self displayNotes]; 
 
}

- (void)displayNotes;
{
    NSArray * letters = [roots componentsSeparatedByString:@","];
        
        if(DR1 > 34 && DR1 < 85)[ drum1Field  setStringValue:[letters objectAtIndex:DR1-35]];
        if(DR2 > 34 && DR2 < 85)[ drum2Field  setStringValue:[letters objectAtIndex:DR2-35]];
        if(DR3 > 34 && DR3 < 85)[ drum3Field  setStringValue:[letters objectAtIndex:DR3-35]];
        if(DR4 > 34 && DR4 < 85)[ drum4Field  setStringValue:[letters objectAtIndex:DR4-35]];
        if(DR5 > 34 && DR5 < 85)[ drum5Field   setStringValue:[letters objectAtIndex:DR5-35] ]; 
        if(DR6 > 34 && DR6 < 85)[ drum6Field   setStringValue:[letters objectAtIndex:DR6-35] ]; 
        if(DR7 > 34 && DR7 < 85)[ drum7Field   setStringValue:[letters objectAtIndex:DR7-35] ]; 
        if(DR8 > 34 && DR8 < 85)[ drum8Field   setStringValue:[letters objectAtIndex:DR8-35] ]; 
        if(DR9 > 34 && DR9 < 85)[ drum9Field   setStringValue:[letters objectAtIndex:DR9-35] ]; 
        if(DR10 > 34 && DR10 < 85)[ drum10Field  setStringValue:[letters objectAtIndex:DR10-35] ]; 
        
        [dr1  setIntValue:DR1-35]; [dr2  setIntValue:DR2-35]; [dr3  setIntValue:DR3-35]; [dr4  setIntValue:DR4-35];
        [dr5  setIntValue:DR5-35]; [dr6  setIntValue:DR6-35]; [dr7  setIntValue:DR7-35]; [dr8  setIntValue:DR8-35];
        [dr9  setIntValue:DR9-35]; [dr10  setIntValue:DR10-35];
        
        if(KIT==1)    [ kitField  setStringValue:@"Piano"];
        if(KIT==5)    [ kitField  setStringValue:@"Rhodes"];
        if(KIT==20)   [ kitField  setStringValue:@"Organ"];
        if(KIT==35)   [ kitField  setStringValue:@"Electric Bass"];
        if(KIT==13)   [ kitField  setStringValue:@"Marimba"];
        if(KIT==11)   [ kitField  setStringValue:@"Music Box"];
        if(KIT==48)   [ kitField  setStringValue:@"Timpani"];
        if(KIT==115)  [ kitField  setStringValue:@"Steel Drum"];
        if(KIT==109)  [ kitField  setStringValue:@"Kalimba"];
        if(KIT==118)  [ kitField  setStringValue:@"Melodic Drum"];
        if(KIT==54)   [ kitField  setStringValue:@"Doot"];
        }

- (void)displayDrums;  
{
NSArray * names = [DRnames componentsSeparatedByString:@","]; 


if(DR1 > 34 && DR1 < 85) [ drum1Field   setStringValue:[names objectAtIndex:DR1-35] ]; 
if(DR2 > 34 && DR2 < 85) [ drum2Field   setStringValue:[names objectAtIndex:DR2-35] ]; 
if(DR3 > 34 && DR3 < 85) [ drum3Field   setStringValue:[names objectAtIndex:DR3-35] ]; 
if(DR4 > 34 && DR4 < 85) [ drum4Field   setStringValue:[names objectAtIndex:DR4-35] ]; 
if(DR5 > 34 && DR5 < 85) [ drum5Field   setStringValue:[names objectAtIndex:DR5-35] ]; 
if(DR6 > 34 && DR6 < 85) [ drum6Field   setStringValue:[names objectAtIndex:DR6-35] ]; 
if(DR7 > 34 && DR7 < 85) [ drum7Field   setStringValue:[names objectAtIndex:DR7-35] ]; 
if(DR8 > 34 && DR8 < 85) [ drum8Field   setStringValue:[names objectAtIndex:DR8-35] ]; 
if(DR9 > 34 && DR9 < 85) [ drum9Field   setStringValue:[names objectAtIndex:DR9-35] ]; 
if(DR10 > 34 && DR10 < 85)[ drum10Field  setStringValue:[names objectAtIndex:DR10-35] ];
 
if(KIT==16417)[ kitField  setStringValue:@"Jazz"];
if(KIT==16401)[ kitField  setStringValue:@"Power"];
if(KIT==16409)[ kitField  setStringValue:@"Electronic"];
if(KIT==16410)[ kitField  setStringValue:@"Analog"];
if(KIT==16433)[ kitField  setStringValue:@"Orchestra"];
if(KIT==16441)[ kitField  setStringValue:@"SFX"];

[dr1  setIntValue:DR1-35]; [dr2  setIntValue:DR2-35]; [dr3  setIntValue:DR3-35]; [dr4  setIntValue:DR4-35];
[dr5  setIntValue:DR5-35]; [dr6  setIntValue:DR6-35]; [dr7  setIntValue:DR7-35]; [dr8  setIntValue:DR8-35];
[dr9  setIntValue:DR9-35]; [dr10  setIntValue:DR10-35];
}

initVOL()
{
    int x;
    
    for(x=0; x < 64; x++){
        DR1VOL[x]   = DV1;
        DR2VOL[x]   = DV2;
        DR3VOL[x]   = DV3;
        DR4VOL[x]   = DV4;
        DR5VOL[x]   = DV5;
        DR6VOL[x]   = DV6;
        DR7VOL[x]   = DV7;
        DR8VOL[x]   = DV8;
        DR9VOL[x]   = DV9;
        DR10VOL[x]  = DV10;
    }
}

setVOL()
{
int x;

for(x=0; x < 64; x++){
if(DEMO==1)DR1VOL[x]   = DV1;
if(DEMO==2)DR2VOL[x]   = DV2;
if(DEMO==3)DR3VOL[x]   = DV3;
if(DEMO==4)DR4VOL[x]   = DV4;
if(DEMO==5)DR5VOL[x]   = DV5;
if(DEMO==6)DR6VOL[x]   = DV6;
if(DEMO==7)DR7VOL[x]   = DV7;
if(DEMO==8)DR8VOL[x]   = DV8;
if(DEMO==9)DR9VOL[x]   = DV9;
if(DEMO==10)DR10VOL[x]  = DV10;
}

}

- (void)displayVolume;
{
if(DV1)[ vol1Field  setIntValue:DV1]; 
else   [ vol1Field  setStringValue:@"Off"];
[ dr1Vel  setIntValue:DV1];

if(DV2)[ vol2Field  setIntValue:DV2]; 
else   [ vol2Field  setStringValue:@"Off"];
[ dr2Vel  setIntValue:DV2];

if(DV3)[ vol3Field  setIntValue:DV3]; 
else   [ vol3Field  setStringValue:@"Off"];
[ dr3Vel  setIntValue:DV3];

if(DV4)[ vol4Field  setIntValue:DV4]; 
else   [ vol4Field  setStringValue:@"Off"];
[ dr4Vel  setIntValue:DV4];

if(DV5)[ vol5Field  setIntValue:DV5]; 
else   [ vol5Field  setStringValue:@"Off"];
[ dr5Vel  setIntValue:DV5];

if(DV6)[ vol6Field  setIntValue:DV6]; 
else   [ vol6Field  setStringValue:@"Off"];
[ dr6Vel  setIntValue:DV6];

if(DV7)[ vol7Field  setIntValue:DV7]; 
else   [ vol7Field  setStringValue:@"Off"];
[ dr7Vel  setIntValue:DV7];

if(DV8)[ vol8Field  setIntValue:DV8]; 
else   [ vol8Field  setStringValue:@"Off"];
[ dr8Vel  setIntValue:DV8];

if(DV9)[ vol9Field  setIntValue:DV9]; 
else   [ vol9Field  setStringValue:@"Off"];
[ dr9Vel  setIntValue:DV9];

if(DV10)[ vol10Field  setIntValue:DV10]; 
else   [ vol10Field  setStringValue:@"Off"];
[ dr10Vel  setIntValue:DV10];

}

#pragma mark PLAY

doDEMO()
{

long d; 
        
        if(DEMO == 1)NAPlayNote(na,nc,DR1, DV1);
        if(DEMO == 2)NAPlayNote(na,nc,DR2, DV2);
        if(DEMO == 3)NAPlayNote(na,nc,DR3, DV3);
        if(DEMO == 4)NAPlayNote(na,nc,DR4, DV4);
        if(DEMO == 5)NAPlayNote(na,nc,DR5, DV5);
        if(DEMO == 6)NAPlayNote(na,nc,DR6, DV6);
        if(DEMO == 7)NAPlayNote(na,nc,DR7, DV7);
        if(DEMO == 8)NAPlayNote(na,nc,DR8, DV8);
        if(DEMO == 9)NAPlayNote(na,nc,DR9, DV9);
        if(DEMO == 10)NAPlayNote(na,nc,DR10, DV10);
        
        Delay( 4, &d );
        
        if(DEMO == 1)NAPlayNote(na,nc,DR1, 0);
        if(DEMO == 2)NAPlayNote(na,nc,DR2, 0);
        if(DEMO == 3)NAPlayNote(na,nc,DR3, 0);
        if(DEMO == 4)NAPlayNote(na,nc,DR4, 0);
        if(DEMO == 5)NAPlayNote(na,nc,DR5, 0);
        if(DEMO == 6)NAPlayNote(na,nc,DR6, 0);
        if(DEMO == 7)NAPlayNote(na,nc,DR7, 0);
        if(DEMO == 8)NAPlayNote(na,nc,DR8, 0);
        if(DEMO == 9)NAPlayNote(na,nc,DR9, 0);
        if(DEMO == 10)NAPlayNote(na,nc,DR10, 0);
        
    } // end if DEMO ==============
    

- (void)respondToTimer:(NSTimer *)timer;

{
long d;  
int T = BARS*BEATS*SUB;
        

    if(DR1BUF[LOOP] && DV1)NAPlayNote(na,nc,DR1, DR1VOL[LOOP]);
    if(DR2BUF[LOOP] && DV2)NAPlayNote(na,nc,DR2, DR2VOL[LOOP]);
    if(DR3BUF[LOOP] && DV3)NAPlayNote(na,nc,DR3, DR3VOL[LOOP]);
    if(DR4BUF[LOOP] && DV4)NAPlayNote(na,nc,DR4, DR4VOL[LOOP]);
    if(DR5BUF[LOOP] && DV5)NAPlayNote(na,nc,DR5, DR5VOL[LOOP]);
    if(DR6BUF[LOOP] && DV6)NAPlayNote(na,nc,DR6, DR6VOL[LOOP]);
    if(DR7BUF[LOOP] && DV7)NAPlayNote(na,nc,DR7, DR7VOL[LOOP]);
    if(DR8BUF[LOOP] && DV8)NAPlayNote(na,nc,DR8, DR8VOL[LOOP]);
    if(DR9BUF[LOOP] && DV9)NAPlayNote(na,nc,DR9, DR9VOL[LOOP]);
    if(DR10BUF[LOOP] && DV10)NAPlayNote(na,nc,DR10, DR10VOL[LOOP]);
    
    TP = (3600/TEMPO)/SUB;       // printf("TP=%i  ",TP);
     
     Delay( TP, &d );
         
    if(DR1BUF[LOOP] && DV1)NAPlayNote(na,nc,DR1, 0);
    if(DR2BUF[LOOP] && DV2)NAPlayNote(na,nc,DR2, 0);
    if(DR3BUF[LOOP] && DV3)NAPlayNote(na,nc,DR3, 0);
    if(DR4BUF[LOOP] && DV4)NAPlayNote(na,nc,DR4, 0);
    if(DR5BUF[LOOP] && DV5)NAPlayNote(na,nc,DR5, 0);
    if(DR6BUF[LOOP] && DV6)NAPlayNote(na,nc,DR6, 0);
    if(DR7BUF[LOOP] && DV7)NAPlayNote(na,nc,DR7, 0);
    if(DR8BUF[LOOP] && DV8)NAPlayNote(na,nc,DR8, 0);
    if(DR9BUF[LOOP] && DV9)NAPlayNote(na,nc,DR9, 0);
    if(DR10BUF[LOOP] && DV10)NAPlayNote(na,nc,DR10, 0);
       
if(++LOOP >= T)LOOP=0; 

 [ count setStringValue:[NSString stringWithFormat:@"%i:%i",(LOOP/SUB)+1,LOOP+1]];
 
 if(INF && LOOP==0){ randomize(); [self setNeedsDisplay:YES]; if(KIT > 1000)[self displayDrums]; else[self displayNotes]; }

}

- (IBAction)setBars:(id)sender
{
BARS = [ [sender selectedCell] tag ]; 
clearBufs(); [self setNeedsDisplay:YES];
}

- (IBAction)setBeats:(id)sender;
{
BEATS = [ [sender selectedCell] tag ]; // printf("beats=%i " ,BEATS);
clearBufs(); [self setNeedsDisplay:YES];
}


- (IBAction)setDiv:(id)sender
{
SUB = [ [sender selectedCell] tag ]; //printf("SUB=%i " ,SUB);
clearBufs(); [self setNeedsDisplay:YES];
}

- (IBAction)setPos:(id)sender; // is now reset
{
  
  oldV1=DV1; oldV2=DV2; oldV3=DV3; oldV4=DV4; oldV5=DV5;
  oldV6=DV6; oldV7=DV7; oldV8=DV8; oldV9=DV9; oldV10=DV10; 

setVOL(); [self displayVolume];
GRID=1; [self setNeedsDisplay:YES];
}

#pragma mark  RANDOM

- (IBAction)doRandom:(id)sender 
{
randTOG = [ [sender selectedCell] tag ];
randomize(); dataTOG=1;
if(randTOG == 1 || randTOG == 4)[self displayVolume];
if(randTOG == 2 || randTOG == 5)if(KIT > 1000)[self displayDrums];  else [self displayNotes];
[undo setTitle:@"Revert"];
[self setNeedsDisplay:YES]; 
}

randomize()
{

int x, r, y, d;

srand((time(NULL) % 100));

if(randTOG > 2){ clearBufs(); setVOL(); } 

for(x=0; x < SUB*BARS; x++)
{

    r = (unsigned int) rand();
    r = abs (( r ) % LNG); 
    if(randTOG > 2)oldD1=DR1BUF[r]=DR1; 
B1: y = (unsigned int) rand();
    y = abs (( y ) % 128);
    if(randTOG == 1 || randTOG  > 3 )oldV1 = DR1VOL[r]=y; if(y < 60)goto B1;
   // printf("\ny=%i ",y);
    
    r = (unsigned int) rand();
    r = abs (( r ) % LNG); // printf("r=%i ",r);
    if(randTOG > 2)oldD2=DR2BUF[r]=DR2; 
B2: y = (unsigned int) rand();
    y = abs (( y ) % 127); if(y < 60)goto B2;
    if(randTOG == 1 || randTOG  > 3)oldV2 = DR2VOL[r]=y;
    
    r = (unsigned int) rand();
    r = abs (( r ) % LNG); // printf("r=%i ",r);
    if(randTOG > 2)oldD3=DR3BUF[r]=DR3; 
B3: y = (unsigned int) rand();
    y = abs (( y ) % 127); if(y < 60)goto B3;
    if(randTOG == 1 || randTOG  > 3)oldV3 = DR3VOL[r]=y;
    
    r = (unsigned int) rand();
    r = abs (( r ) % LNG); // printf("r=%i ",r);
    if(randTOG > 2)oldD4=DR4BUF[r]=DR4; 
B4: y = (unsigned int) rand();
    y = abs (( y ) % 127); if(y < 60)goto B4;
    if(randTOG == 1 || randTOG  > 3)oldV4 = DR4VOL[r]=y;
    
    r = (unsigned int) rand();
    r = abs (( r ) % LNG); // printf("r=%i ",r);
    if(randTOG > 2)oldD5=DR5BUF[r]=DR5; 
B5: y = (unsigned int) rand();
    y = abs (( y ) % 127); if(y < 60)goto B5;
    if(randTOG == 1 || randTOG  > 3)oldV5 = DR5VOL[r]=y;
    
    r = (unsigned int) rand();
    r = abs (( r ) % LNG); // printf("r=%i ",r);
    if(randTOG > 2)oldD6=DR6BUF[r]=DR6;
B6: y = (unsigned int) rand();
    y = abs (( y ) % 127); if(y < 60)goto B6;
    if(randTOG == 1 || randTOG  > 3)oldV6 = DR6VOL[r]=y;
    
    r = (unsigned int) rand();
    r = abs (( r ) % LNG); // printf("r=%i ",r);
    if(randTOG > 2)oldD7=DR7BUF[r]=DR7; 
B7: y = (unsigned int) rand();
    y = abs (( y ) % 127); if(y < 60)goto B7;
    if(randTOG == 1 || randTOG  > 3)oldV7 = DR7VOL[r]=y;
    
    r = (unsigned int) rand();
    r = abs (( r ) % LNG); // printf("r=%i ",r);
    if(randTOG > 2)oldD8=DR8BUF[r]=DR8; 
B8: y = (unsigned int) rand();
    y = abs (( y ) % 127); if(y < 60)goto B8;
    if(randTOG == 1 || randTOG  > 3)oldV8 = DR8VOL[r]=y;
    
    r = (unsigned int) rand();
    r = abs (( r ) % LNG); // printf("r=%i ",r);
    if(randTOG > 2)oldD9=DR9BUF[r]=DR9; 
B9: y = (unsigned int) rand();
    y = abs (( y ) % 127); if(y < 60)goto B9;
    if(randTOG == 1 || randTOG  > 3)oldV9 = DR9VOL[r]=y;

    r = (unsigned int) rand();
    r = abs (( r ) % LNG); // printf("r=%i ",r);
    if(randTOG > 2)oldD10=DR10BUF[r]=DR10; 
B10:y = (unsigned int) rand();
    y = abs (( y ) % 127); if(y < 60)goto B10;
    if(randTOG == 1 || randTOG > 3)oldV10 = DR10VOL[r]=y;
    
   }
   
   if(randTOG == 2 || randTOG == 5)
   {
    r = (unsigned int) rand();
    r = abs (( r ) % 48); //printf("r=%i ",r);
    DR1 = r + 35;
    
    r = (unsigned int) rand();
    r = abs (( r ) % 48); //printf("r=%i ",r);
    DR2 = r + 35;
    
    r = (unsigned int) rand();
    r = abs (( r ) % 48); //printf("r=%i ",r);
    DR3 = r + 35;
    
    r = (unsigned int) rand();
    r = abs (( r ) % 48);// printf("r=%i ",r);
    DR4 = r + 35;
    
    r = (unsigned int) rand();
    r = abs (( r ) % 48); //printf("r=%i ",r);
    DR6 = r + 35;
    
    r = (unsigned int) rand();
    r = abs (( r ) % 48);// printf("r=%i ",r);
    DR7 = r + 35;
    
    r = (unsigned int) rand();
    r = abs (( r ) % 48);// printf("r=%i ",r);
    DR8 = r + 35;
    
    r = (unsigned int) rand();
    r = abs (( r ) % 48); //printf("r=%i ",r);
    DR9 = r + 35;
    
    r = (unsigned int) rand();
    r = abs (( r ) % 48);// printf("r=%i ",r);
    DR10 = r + 35;
    }     
}


#pragma mark SET VELOCITY ================================

- (IBAction)setD1vol:(id)sender
{
oldV1=DV1; 
DV1 = [ dr1Vel intValue ]; 
if(DV1)[ vol1Field  setIntValue:DV1]; 
else   [ vol1Field  setStringValue:@"Off"];
}

- (IBAction)setD2vol:(id)sender
{
DV2 = [ dr2Vel intValue ];  [ vol2Field  setIntValue:DV2];  // printf("DV2=%i " ,DV2);
if(DV2)[ vol2Field  setIntValue:DV2]; 
else   [ vol2Field  setStringValue:@"Off"];
}

- (IBAction)setD3vol:(id)sender
{
DV3 = [ dr3Vel intValue ];  [ vol3Field  setIntValue:DV3]; //printf("DV3=%i " ,DV3);
if(DV3)[ vol3Field  setIntValue:DV3]; 
else   [ vol3Field  setStringValue:@"Off"];
}

- (IBAction)setD4vol:(id)sender
{
DV4 = [ dr4Vel intValue ];  [ vol4Field  setIntValue:DV4]; //printf("DV4=%i " ,DV4);
if(DV4)[ vol4Field  setIntValue:DV4]; 
else   [ vol4Field  setStringValue:@"Off"];
}

- (IBAction)setD5vol:(id)sender
{
DV5 = [ dr5Vel intValue ];  [ vol5Field  setIntValue:DV5];// printf("DV5=%i " ,DV5);
if(DV5)[ vol5Field  setIntValue:DV5]; 
else   [ vol5Field  setStringValue:@"Off"];
}

- (IBAction)setD6vol:(id)sender;
{
DV6 = [ dr6Vel intValue ]; printf("DV6=%i " ,DV6);
if(DV6)[ vol6Field  setIntValue:DV6]; 
else   [ vol6Field  setStringValue:@"Off"];
}

- (IBAction)setD7vol:(id)sender
{
DV7 = [ dr7Vel intValue ];  [ vol7Field  setIntValue:DV7];    // printf("DV5=%i " ,DV5);
if(DV7)[ vol7Field  setIntValue:DV7]; 
else   [ vol7Field  setStringValue:@"Off"];
}

- (IBAction)setD8vol:(id)sender
{
DV8 = [ dr8Vel intValue ];  [ vol8Field  setIntValue:DV8];   // printf("DV5=%i " ,DV5);
if(DV8)[ vol8Field  setIntValue:DV8]; 
else   [ vol8Field  setStringValue:@"Off"];
}

- (IBAction)setD9vol:(id)sender
{
DV9 = [ dr9Vel intValue ];  [ vol9Field  setIntValue:DV9];   // printf("DV5=%i " ,DV5);
if(DV9)[ vol9Field  setIntValue:DV9]; 
else   [ vol9Field  setStringValue:@"Off"];
}

- (IBAction)setD10vol:(id)sender
{
DV10 = [ dr10Vel intValue ];  [ vol10Field  setIntValue:DV10];   // printf("DV5=%i " ,DV5);
if(DV10)[ vol10Field  setIntValue:DV10]; 
else   [ vol10Field  setStringValue:@"Off"];
}


#pragma mark SET DRUMS  ===================================


- (IBAction)setDrum1:(id)sender
{
 DR1 = [ dr1 intValue ] + 35;  //  printf("DR1=%i " ,DR1);  
 if(KIT > 1000)[self displayDrums];  else [self displayNotes]; 
}

- (IBAction)setDrum2:(id)sender
{
DR2 = [ dr2 intValue ] + 35;  [ drum2Field  setIntValue:DR2];  
if(KIT > 1000)[self displayDrums];  else [self displayNotes];
}

- (IBAction)setDrum3:(id)sender
{
DR3 = [ dr3 intValue ] + 35;  [ drum3Field  setIntValue:DR3];  
if(KIT > 1000)[self displayDrums];  else [self displayNotes];
}

- (IBAction)setDrum4:(id)sender
{
DR4 = [ dr4 intValue ] + 35;  [ drum4Field  setIntValue:DR4];  DRS=4;
if(KIT > 1000)[self displayDrums];  else [self displayNotes];
}

- (IBAction)setDrum5:(id)sender
{
DR5 = [ dr5 intValue ] + 35;  [ drum5Field  setIntValue:DR5];  DRS=5;
if(KIT > 1000)[self displayDrums];  else [self displayNotes];
}

- (IBAction)setDrum6:(id)sender
{
DR6 = [ dr6 intValue ] + 35;  [ drum6Field  setIntValue:DR6];
if(KIT > 1000)[self displayDrums];  else [self displayNotes];
}

- (IBAction)setDrum7:(id)sender
{
DR7 = [ dr7 intValue ] + 35;  [ drum7Field  setIntValue:DR7];
if(KIT > 1000)[self displayDrums];  else [self displayNotes];
}


- (IBAction)setDrum8:(id)sender
{
DR8 = [ dr8 intValue ] + 35;  [ drum8Field  setIntValue:DR8];
if(KIT > 1000)[self displayDrums];  else [self displayNotes];
}

- (IBAction)setDrum9:(id)sender
{
DR9 = [ dr9 intValue ] + 35;  [ drum9Field  setIntValue:DR9];
if(KIT > 1000)[self displayDrums];  else [self displayNotes];
}

- (IBAction)setDrum10:(id)sender
{
DR10= [ dr10 intValue ] + 35;  [ drum10Field  setIntValue:DR10];
if(KIT > 1000)[self displayDrums];  else [self displayNotes];
}

// END SET DRUMS  ===========================================================

- (IBAction)setKit:(id)sender
{
oldKIT = KIT;
KIT  = [ [sender selectedCell] tag ]; 
initQT();   //printf("KIT=%i " ,KIT);
if(KIT > 1000)[self displayDrums];  else [self displayNotes];
}

- (IBAction)setTempo:(id)sender
{
  TEMPO = [ tempoSlider intValue ];  [ tempoField  setIntValue:TEMPO]; 
}

- (IBAction)setTest:(id)sender
{
DEMO = [ [sender selectedCell] tag ];  //printf("DEMO=%i " ,DEMO);
 doDEMO(); 
}

#pragma mark PRESETS

- (IBAction)doPreset:(id)sender;
{
int x;

PRE = [ [sender selectedCell] tag ];
clearBufs(); dataTOG=1;

if(PRE==1){
BARS=2; SUB=3; BEATS=4; DR1=51; DR2=42; DR3=37; DR6=63; DV1=80; KIT = 16417;

for(x=0; x < 24; x++){
DR1BUF[x] = SOUL[0][x];
DR2BUF[x] = SOUL[1][x];
DR3BUF[x] = SOUL[2][x];
DR6BUF[x] = SOUL[3][x]; ;
}
TEMPO=100;  [ fileField  setStringValue:@"JAZZ 4/4"]; 
}

if(PRE==2){
BARS=2; SUB=3; BEATS=3; DR1=51; DR2=42; DR3=37; DR4=36; DR8=36; DV1=60; KIT = 16417;

for(x=0; x < 18; x++){
DR1BUF[x] = JAZZ3[0][x];
DR2BUF[x] = JAZZ3[1][x];
DR3BUF[x] = JAZZ3[2][x];
DR8BUF[x] = JAZZ3[3][x];;
}
TEMPO=100;  [ fileField  setStringValue:@"JAZZ 3/4"];
}

if(PRE==3){
BARS=2; SUB=4; BEATS=4; clearBufs(); DR1=42; DR2=38; DR3=54; DR6=35; DR8=36; DV8=90; KIT = 16401;

for(x=0; x < 32; x++){
DR1BUF[x] = FUNK1[0][x];
DR2BUF[x] = FUNK1[2][x];
DR3BUF[x] = FUNK1[1][x];
DR8BUF[x] = FUNK1[3][x];;
}
TEMPO=80;  [ fileField  setStringValue:@"FUNK 1"];
}

if(PRE==4){
BARS=2; SUB=2; BEATS=4; clearBufs(); DR1=82; DR2=69; DR3=37; DR8=36; KIT = 16417;

for(x=0; x < 16; x++){
DR1BUF[x] = BOSSA[0][x];
DR2BUF[x] = BOSSA[1][x];
DR3BUF[x] = BOSSA[2][x];
DR8BUF[x] = BOSSA[3][x];;
}
TEMPO=160; [ fileField  setStringValue:@"JAZZ BOSSA"];
}

if(PRE==5){
BARS=2; SUB=2; BEATS=4; clearBufs();  DR1=82; DR2=37; DR4=67; DR8=36; KIT = 16417;

for(x=0; x < 16; x++){
DR1BUF[x] = SALSA[0][x];
DR2BUF[x] = SALSA[2][x];
DR4BUF[x] = SALSA[1][x];  
DR8BUF[x] = SALSA[3][x];  
}
TEMPO=160; [ fileField  setStringValue:@"SALSA"];
}

if(PRE==6){
BARS=2; SUB=2; BEATS=3; clearBufs(); DR1=53; DR2=37; DR3=44; DR8=36; DV1=60; DV8=100; KIT = 16417;

for(x=0; x < 12; x++){
DR1BUF[x] = LATIN3[0][x];
DR2BUF[x] = LATIN3[1][x];
DR3BUF[x] = LATIN3[2][x];
DR8BUF[x] = LATIN3[3][x];
}
TEMPO=130; [ fileField  setStringValue:@"LATIN 3/4"];
}

if(PRE==7){
BARS=2; SUB=2; BEATS=5; clearBufs(); DR1=53; DR2=63; DR3=56; DR8=36; DV2=100; DV3=70; KIT = 16417;

for(x=0; x < 20; x++){
DR1BUF[x] = LATIN5[0][x];
DR2BUF[x] = LATIN5[1][x];
DR3BUF[x] = LATIN5[2][x];
DR8BUF[x] = LATIN5[3][x];
}
TEMPO=160; [ fileField  setStringValue:@"LATIN 5/4"];
}

if(PRE==8){
BARS=2; SUB=4; BEATS=4; DR1=59; DR2=42; DR3=75; DR8=36; DV1=90; DV3=100; KIT = 16417;

for(x=0; x < 32; x++){
DR1BUF[x] = ECM[0][x];
DR2BUF[x] = ECM[1][x];
DR3BUF[x] = ECM[2][x];
DR8BUF[x] = ECM[3][x];  
}
TEMPO=100; [ fileField  setStringValue:@"ECM"];
}

if(PRE==9){
BARS=1; SUB=2; BEATS=4; DR1=56; DR2=37;  KIT = 16417; 

DR1BUF[0] = 1;
DR2BUF[2] = 1;
DR2BUF[4] = 1;
DR2BUF[6] = 1; 
TEMPO=120; [ fileField  setStringValue:@"CLICK TRACK"];
}

if(PRE==10){
BARS=2; SUB=2; BEATS=4;
DR1=60; DR2=63; DR3=65; DR4=67; DR5=70; DR6=72; DR7=36; DR8=43; DR9=58; DR10=60;  KIT = 109; 
TEMPO=160; [ fileField  setStringValue:@"PENTATONIC"];
randTOG=4; randomize();
}

if(PRE==11){
BARS=2; SUB=2; BEATS=4;
DR1=60; DR2=62; DR3=67; DR4=72; DR5=74; DR6=79; DR7=36; DR8=43; DR9=60; DR10=62;  KIT = 5; 
TEMPO=160; [ fileField  setStringValue:@"SUS 2"];
randTOG=4; randomize();
}

if(PRE==12){
BARS=2; SUB=2; BEATS=4;
DR1=56; DR2=58; DR3=63; DR4=67; DR5=65; DR6=70; DR7=41; DR8=48; DR9=55; DR10=77;  KIT = 1; 
TEMPO=160; [ fileField  setStringValue:@"MIN 11"];
randTOG=4; randomize();
}

[ tempoField  setIntValue:TEMPO]; [ tempoSlider  setIntValue:TEMPO];

setVOL(); [self displayVolume];if(KIT > 1000)[self displayDrums];  else [self displayNotes];
[self setNeedsDisplay:YES]; 

}


- (IBAction)saveDrums:(id)sender;
{
saveSET=1; [self saveFile:self];
}

- (IBAction)openDrums:(id)sender;
{
saveSET=1; [self openFile:self];
}

- (IBAction)saveMIDI:(id)sender;
{
exportMIDI=1;
[self saveFile:self];
}

- (IBAction)fixQT:(id)sender;
{
 CloseComponent(na); NADisposeNoteChannel(na,nc); initQT(); 
NSRunAlertPanel(@"Possible QT Problem",@"Attemped a Quicktime Fix.\n\nIf QT still does not respond, Save your project...\nQuit the App and Relaunch.",@"I Will",nil,nil);
}


#pragma mark SAVE FILES

- (IBAction)saveFile:(id)sender;
{

    NSString *fileName; NSString *dpath;

    if(exportMIDI)
    {
   // exportMIDI=0; NSRunAlertPanel(@"MIDI Export",@"Working on it – soon to be available...",@"Great!",nil,nil); return;
    dpath = @"~/Desktop/DrumThing/Saves/MIDI Files";
    fileName = @"Groove1.mid";
    }

    if(saveSET)
    {
    dpath = @"~/Desktop/DrumThing/Saves/DrumSets";
    fileName =   @"DrumSet1";
    }
    
    if(exportMIDI == 0 || saveSET == 0)
    {
NSRunAlertPanel(@"SORRY",@"You will have to purchase this app\nin order to save files",@"I Will, $15 is CHEAP!!",nil,nil); return;
   dpath = @"~/Desktop/DrumThing/Saves/Grooves";
   fileName = lastFile; 
    }
    
    NSSavePanel * savePanel = [NSSavePanel savePanel];
    SEL sel = @selector(savePanelDidEnd:returnCode:contextInfo:);
    [savePanel beginSheetForDirectory:dpath file:fileName modalForWindow:[self window] modalDelegate:self didEndSelector:sel contextInfo:nil];
}

- (void)savePanelDidEnd:(NSSavePanel *)sheet returnCode:(int)returnCode contextInfo:(void *)context
{

    if (returnCode == 1);
{

if (returnCode == 0)return;

    dataTOG=0;

    NSString * name = [sheet filename]; 
    name = [name lastPathComponent];     [ fileField  setStringValue:name];
    
if(saveSET)
{
                              
NSMutableArray * set = [[NSMutableArray alloc] init];
    

    [set addObject:[NSNumber numberWithInt:DR1]];
    [set addObject:[NSNumber numberWithInt:DR2]];
    [set addObject:[NSNumber numberWithInt:DR3]];
    [set addObject:[NSNumber numberWithInt:DR4]];
    [set addObject:[NSNumber numberWithInt:DR5]];
    [set addObject:[NSNumber numberWithInt:DR6]];
    [set addObject:[NSNumber numberWithInt:DR7]];
    [set addObject:[NSNumber numberWithInt:DR8]];
    [set addObject:[NSNumber numberWithInt:DR9]];
    [set addObject:[NSNumber numberWithInt:DR10]];
    [set addObject:[NSNumber numberWithInt:KIT]];
    
    [set writeToFile:[sheet filename] atomically:YES];
   
    [set release]; 

    saveSET=0; return;

}
    
} // end if(saveset)---

#pragma mark MIDI SAVE
    
    if(exportMIDI)
    {
    
        
        int x; NSArray * names = [DRnames componentsSeparatedByString:@","]; 
   
        MusicSequence	musicSequence;	
        if (NewMusicSequence(&musicSequence) != noErr)
            [NSException raise:@"main" format:@"Cannot create music sequence."];
        
        MusicTrack tempoTrack;
        if (MusicSequenceGetTempoTrack(musicSequence, &tempoTrack) != noErr)
            [NSException raise:@"main" format:@"Cannot get tempo track."];
        
        if (MusicTrackNewExtendedTempoEvent(tempoTrack, 0.0, TEMPO*2) != noErr)
            [NSException raise:@"main" format:@"Cannot add tempo event."];
        
        MusicTrack musicTrack;
        if (MusicSequenceNewTrack(musicSequence, &musicTrack) != noErr)
            [NSException raise:@"main" format:@"Cannot create music track."];
        
       // NSString * trName; 
       // [trName setStringValue:[names objectAtIndex:DR1-35]];
        char *trackName = "DRUM1";  //[names objectAtIndex:DR1-35]; //trName; // = "DRUM1";
       

        MIDIMetaEvent *trackNameMetaEvent = CFAllocatorAllocate(NULL, sizeof(MIDIMetaEvent) - 1 + strlen(trackName), 0);
        trackNameMetaEvent->metaEventType = 0x03;
        trackNameMetaEvent->dataLength = strlen(trackName);
        strncpy(trackNameMetaEvent->data, trackName, strlen(trackName));
        
        if (MusicTrackNewMetaEvent(musicTrack, 0.0, trackNameMetaEvent) != noErr)
            [NSException raise:@"main" format:@"Cannot add track name meta event."];
        
        const int channel = 9;
        MIDIChannelMessage patchChange = { 0xc0 | channel, 33 - 1, 0, 0 };
        if (MusicTrackNewMIDIChannelEvent(musicTrack, 0.0, &patchChange) != noErr)
            [NSException raise:@"main" format:@"Cannot add patch change channel event."];
        
        MIDINoteMessage note;
        note.channel = channel;
        note.velocity = 120.00;
        note.duration = 1.2;
        
        // Add Drum1 data
        for (x = 0; x < 16; x++)
        {
            if(DR1BUF[x]){
                note.note = DR1;// printf("DR1=%i\n",DR1);
                if (MusicTrackNewMIDINoteEvent(musicTrack, x * 1.0, &note) != noErr)
                [NSException raise:@"main" format:@"Cannot add note to track."];
            }
            
            if(DR2BUF[x]){
                note.note = DR2;// printf("DR1=%i\n",DR1);
                if (MusicTrackNewMIDINoteEvent(musicTrack, x * 1.0, &note) != noErr)
                    [NSException raise:@"main" format:@"Cannot add note to track."];
            }
            
            if(DR3BUF[x]){
                note.note = DR3;// printf("DR1=%i\n",DR1);
                if (MusicTrackNewMIDINoteEvent(musicTrack, x * 1.0, &note) != noErr)
                    [NSException raise:@"main" format:@"Cannot add note to track."];
            }
            
            if(DR4BUF[x]){
                note.note = DR3;
                if (MusicTrackNewMIDINoteEvent(musicTrack, x * 1.0, &note) != noErr)
                    [NSException raise:@"main" format:@"Cannot add note to track."];
            }
            
            if(DR5BUF[x]){
                note.note = DR5;
                if (MusicTrackNewMIDINoteEvent(musicTrack, x * 1.0, &note) != noErr)
                    [NSException raise:@"main" format:@"Cannot add note to track."];
            }
            
            
            if(DR6BUF[x]){
                note.note = DR6;
                if (MusicTrackNewMIDINoteEvent(musicTrack, x * 1.0, &note) != noErr)
                    [NSException raise:@"main" format:@"Cannot add note to track."];
            }
            
            if(DR7BUF[x]){
                note.note = DR7;
                if (MusicTrackNewMIDINoteEvent(musicTrack, x * 1.0, &note) != noErr)
                    [NSException raise:@"main" format:@"Cannot add note to track."];
            }
            
            
            if(DR8BUF[x]){
                note.note = DR8;
                if (MusicTrackNewMIDINoteEvent(musicTrack, x * 1.0, &note) != noErr)
                    [NSException raise:@"main" format:@"Cannot add note to track."];
            }
            
            
            if(DR9BUF[x]){
                note.note = DR9;
                if (MusicTrackNewMIDINoteEvent(musicTrack, x * 1.0, &note) != noErr)
                    [NSException raise:@"main" format:@"Cannot add note to track."];
            }
            
            if(DR10BUF[x]){
                note.note = DR10;
                if (MusicTrackNewMIDINoteEvent(musicTrack, x * 1.0, &note) != noErr)
                    [NSException raise:@"main" format:@"Cannot add note to track."];
            }
            
        }
        
        MIDIMetaEvent metaEvent = { 0x2f, 0, 0, 0, 0, { 0 } };
        if (MusicTrackNewMetaEvent(musicTrack, 17.0, &metaEvent) != noErr)
            [NSException raise:@"main" format:@"Cannot add end of track meta event to track."];
        
       // CAShow(musicSequence); // ? log data
        
        CFDataRef dataRef;
        if (MusicSequenceSaveSMFData(musicSequence, &dataRef, 480) != noErr)
            [NSException raise:@"main" format:@"Cannot save SMF data."];
            
            [dataRef writeToFile:[sheet filename] atomically:YES];
        
        

    exportMIDI=0; return;
    }
    
    // Save DATA ===========================================================
    
    int s; int L = SUB*BARS*BEATS; 
    	          
    NSMutableArray * midi = [[NSMutableArray alloc] init];

    [midi addObject:[NSNumber numberWithInt:333]];   // file ID 
    [midi addObject:[NSNumber numberWithInt:SUB]];
    [midi addObject:[NSNumber numberWithInt:BEATS]];
    [midi addObject:[NSNumber numberWithInt:BARS]];
    [midi addObject:[NSNumber numberWithInt:TEMPO]];
    [midi addObject:[NSNumber numberWithInt:KIT]];
    
    [midi addObject:[NSNumber numberWithInt:DR1]];
    [midi addObject:[NSNumber numberWithInt:DR2]];
    [midi addObject:[NSNumber numberWithInt:DR3]];
    [midi addObject:[NSNumber numberWithInt:DR4]];
    [midi addObject:[NSNumber numberWithInt:DR5]];
    [midi addObject:[NSNumber numberWithInt:DR6]];
    [midi addObject:[NSNumber numberWithInt:DR7]];
    [midi addObject:[NSNumber numberWithInt:DR8]];
    [midi addObject:[NSNumber numberWithInt:DR9]];
    [midi addObject:[NSNumber numberWithInt:DR10]];
   
    [midi addObject:[NSNumber numberWithInt:DV1]];
    [midi addObject:[NSNumber numberWithInt:DV2]];
    [midi addObject:[NSNumber numberWithInt:DV3]];
    [midi addObject:[NSNumber numberWithInt:DV4]];
    [midi addObject:[NSNumber numberWithInt:DV5]];
    [midi addObject:[NSNumber numberWithInt:DV6]];
    [midi addObject:[NSNumber numberWithInt:DV7]];
    [midi addObject:[NSNumber numberWithInt:DV8]];
    [midi addObject:[NSNumber numberWithInt:DV9]];
    [midi addObject:[NSNumber numberWithInt:DV10]];
            
     for(s=0; s < 60; s++)
     {
      [midi addObject:[NSNumber numberWithInt:DR1BUF[s]]];
      [midi addObject:[NSNumber numberWithInt:DR1VOL[s]]];
      [midi addObject:[NSNumber numberWithInt:DR2BUF[s]]];
      [midi addObject:[NSNumber numberWithInt:DR2VOL[s]]];
      [midi addObject:[NSNumber numberWithInt:DR3BUF[s]]];
      [midi addObject:[NSNumber numberWithInt:DR3VOL[s]]];
      [midi addObject:[NSNumber numberWithInt:DR4BUF[s]]];
      [midi addObject:[NSNumber numberWithInt:DR4VOL[s]]];
      [midi addObject:[NSNumber numberWithInt:DR5BUF[s]]];
      [midi addObject:[NSNumber numberWithInt:DR5VOL[s]]];
      [midi addObject:[NSNumber numberWithInt:DR6BUF[s]]];
      [midi addObject:[NSNumber numberWithInt:DR6VOL[s]]];
      [midi addObject:[NSNumber numberWithInt:DR7BUF[s]]];
      [midi addObject:[NSNumber numberWithInt:DR7VOL[s]]];
      [midi addObject:[NSNumber numberWithInt:DR8BUF[s]]];
      [midi addObject:[NSNumber numberWithInt:DR8VOL[s]]];
      [midi addObject:[NSNumber numberWithInt:DR9BUF[s]]];
      [midi addObject:[NSNumber numberWithInt:DR9VOL[s]]];
      [midi addObject:[NSNumber numberWithInt:DR10BUF[s]]];
      [midi addObject:[NSNumber numberWithInt:DR10VOL[s]]];
      }
    
   [midi writeToFile:[sheet filename] atomically:YES];
   
   [midi release]; 
   
    
  }  //  END if (returnCode
     
#pragma mark OPEN FILES

- (IBAction)openFile:(id)sender;
{ 

NSString *dpath = @"~/Desktop/GrooveLoops/Saves";
NSString *fileName =  @"Groove1";

NSOpenPanel * openPanel = [NSOpenPanel openPanel];
SEL sel = @selector(openPanelDidEnd:returnCode:contextInfo:);
[openPanel beginSheetForDirectory:dpath file:fileName modalForWindow:[self window] modalDelegate:self   didEndSelector:sel contextInfo:nil];
    
}

- (void)openPanelDidEnd:(NSOpenPanel *)sheet
                           returnCode:(int)returnCode
                                   contextInfo:(void *)context
{    
    if (returnCode == NSOKButton)
    {
    
    int s;  NSNumber *data;
    NSString * name = [sheet filename]; 
    name = [name lastPathComponent]; 
    lastFile = name;

if(saveSET)
{
 NSMutableArray * set = [[NSMutableArray alloc] init];
 
 [set initWithContentsOfFile:[sheet filename] ]; // printf("\nfileName=%s ",[[sheet filename] UTF8String]);
     
    data = [set objectAtIndex:0];  DR1  = [data intValue]; //printf("\nDR1=%i ", DR1);
    data = [set objectAtIndex:1];  DR2  = [data intValue]; //printf("\nDR2=%i ", DR2);
    data = [set objectAtIndex:2];  DR3  = [data intValue]; //printf("\nDR3=%i ", DR3);
    data = [set objectAtIndex:3];  DR4  = [data intValue]; //printf("\nDR4=%i ", DR4);
    data = [set objectAtIndex:4];  DR5  = [data intValue];
    data = [set objectAtIndex:5];  DR6  = [data intValue];
    data = [set objectAtIndex:6];  DR7  = [data intValue];
    data = [set objectAtIndex:7];  DR8  = [data intValue];
    data = [set objectAtIndex:8];  DR9  = [data intValue];
    data = [set objectAtIndex:9];  DR10 = [data intValue];
    data = [set objectAtIndex:10];  KIT = [data intValue]; // printf("\nKIT=%i ", KIT);
        
 [ drSET setStringValue:name];

 if(KIT > 1000)[self displayDrums];  else [self displayNotes]; initQT();

 [set release];  
 

saveSET=0; return;

}

#pragma mark Load drum data
        
    NSMutableArray * midi = [[NSMutableArray alloc] init];

    [midi initWithContentsOfFile:[sheet filename] ]; // printf("\nfileName=%s ",[[sheet filename] UTF8String]);
     
    data = [midi objectAtIndex:0];                          
    int ID = [data intValue];   // printf("\nID=%i ", ID); 
    if(ID != 333){ NSRunAlertPanel(@"INCOMPATABLE FILE",@"Try Again",@"Man!",nil,nil); return; }
    
    data = [midi objectAtIndex:1];
    SUB = [data intValue]; // printf("\nSUB=%i ", SUB);
    
    data = [midi objectAtIndex:2];
    BEATS = [data intValue]; // printf("  BEATS=%i ", BEATS);
    
    data = [midi objectAtIndex:3];
    BARS = [data intValue]; // printf("  BARS=%i ", BARS);
    
    data = [midi objectAtIndex:4];
    TEMPO = [data intValue];   [ tempoField  setIntValue:TEMPO]; [ tempoSlider  setIntValue:TEMPO]; 
        
    data = [midi objectAtIndex:5];
    KIT = [data intValue]; // printf("  KIT=%i ", KIT);
    
    // drum data
    
    data = [midi objectAtIndex:6];
    DR1 = [data intValue];   // printf("  DR1=%i ", DR1);
    
    data = [midi objectAtIndex:7];
    DR2 = [data intValue]; 
    
    data = [midi objectAtIndex:8];
    DR3 = [data intValue]; 
    
    data = [midi objectAtIndex:9];
    DR4 = [data intValue]; 
    
    data = [midi objectAtIndex:10];
    DR5 = [data intValue]; 
    
    data = [midi objectAtIndex:11];
    DR6 = [data intValue];
    
    data = [midi objectAtIndex:12];
    DR7 = [data intValue]; 
    
    data = [midi objectAtIndex:13];
    DR8 = [data intValue];
    
    data = [midi objectAtIndex:14];
    DR9 = [data intValue];
    
    data = [midi objectAtIndex:15];
    DR10 = [data intValue];
    
    // velocity -----------------------
    
     data = [midi objectAtIndex:16];
     DV1 = [data intValue]; //  printf("  DV1=%i ", DV1);
     
     data = [midi objectAtIndex:17];
     DV2 = [data intValue];
    
     data = [midi objectAtIndex:18];
     DV3 = [data intValue];
    
     data = [midi objectAtIndex:19];
     DV4 = [data intValue];
    
     data = [midi objectAtIndex:20];
     DV5 = [data intValue];
    
     data = [midi objectAtIndex:21];
     DV6 = [data intValue];
    
     data = [midi objectAtIndex:22];
     DV7 = [data intValue];
    
     data = [midi objectAtIndex:23];
     DV8 = [data intValue];
     
     data = [midi objectAtIndex:24];
     DV9 = [data intValue];
     
     data = [midi objectAtIndex:25];
     DV10 = [data intValue];
     
     int T = SUB*BEATS*BARS;       // printf("\nT=%i ", T);
     int L = ([midi count]/T)-26;  // printf("  L=%i ", L);
     
     int t=26;

        for(s=0; s < 60; s++){
          
         data = [midi objectAtIndex:s+t];  t++;
         DR1BUF[s] = [data intValue];
         data = [midi objectAtIndex:s+t];  t++;
         DR1VOL[s] = [data intValue];
         
         data = [midi objectAtIndex:s+t];  t++;
         DR2BUF[s] = [data intValue];
         data = [midi objectAtIndex:s+t];  t++;
         DR2VOL[s] = [data intValue];
         
         data = [midi objectAtIndex:s+t];  t++;
         DR3BUF[s] = [data intValue];
         data = [midi objectAtIndex:s+t];  t++;
         DR3VOL[s] = [data intValue];
         
         data = [midi objectAtIndex:s+t];  t++;
         DR4BUF[s] = [data intValue];
         data = [midi objectAtIndex:s+t];  t++;
         DR4VOL[s] = [data intValue];
         
         data = [midi objectAtIndex:s+t];  t++;
         DR5BUF[s] = [data intValue];
         data = [midi objectAtIndex:s+t];  t++;
         DR5VOL[s] = [data intValue];
         
         data = [midi objectAtIndex:s+t];  t++;
         DR6BUF[s] = [data intValue];
         data = [midi objectAtIndex:s+t];  t++;
         DR6VOL[s] = [data intValue];
         
         data = [midi objectAtIndex:s+t];  t++;
         DR7BUF[s] = [data intValue];
         data = [midi objectAtIndex:s+t];  t++;
         DR7VOL[s] = [data intValue];
         
         data = [midi objectAtIndex:s+t];  t++;
         DR8BUF[s] = [data intValue];
         data = [midi objectAtIndex:s+t];  t++;
         DR8VOL[s] = [data intValue];
         
         data = [midi objectAtIndex:s+t];  t++;
         DR9BUF[s] = [data intValue];
         data = [midi objectAtIndex:s+t];  t++;
         DR9VOL[s] = [data intValue];
         
         data = [midi objectAtIndex:s+t]; t++;
         DR10BUF[s] = [data intValue];
         data = [midi objectAtIndex:s+t];  
         DR10VOL[s] = [data intValue]; // printf("\ns=%i t=%i ",s,t);
        
        }
 
 
    [midi release];     
 
 [self displayVolume]; if(KIT > 1000)[self displayDrums];  else [self displayNotes];
 
[ fileField  setStringValue:name];  [ drSET setStringValue:name]; initQT();

GRID=1; [self setNeedsDisplay:YES];

}
}

- (IBAction)cleanBufs:(id)sender;
{
    if(undoTOG==0){ [undo setTitle:@"Revert"]; [self undoClear:self]; }
    else          { [undo setTitle:@"Undo"]; clearBufs(); }
    [self setNeedsDisplay:YES]; if(KIT > 1000)[self displayDrums];  else [self displayNotes];
}

int clearBufs()
{
int x;

undoTOG=1;  //if(undoTOG)return;

for(x=0; x < 64; x++){
DR1undo[x] = DR1BUF[x];
DR2undo[x] = DR2BUF[x];
DR3undo[x] = DR3BUF[x];
DR4undo[x] = DR4BUF[x];
DR5undo[x] = DR5BUF[x];
DR6undo[x] = DR6BUF[x];
DR7undo[x] = DR7BUF[x];
DR8undo[x] = DR8BUF[x];
DR9undo[x] = DR9BUF[x];
DR10undo[x] = DR10BUF[x];
}
for(x=0; x< 64; x++)
DR1BUF[x]=DR2BUF[x]=DR3BUF[x]=DR4BUF[x]=DR5BUF[x]=DR6BUF[x]=DR7BUF[x]=DR8BUF[x]=DR9BUF[x]=DR10BUF[x]=0;

oldD1=DR1; oldD2=DR2; oldD3=DR3; oldD4=DR4; oldD5=DR5; oldD6=DR6; oldD7=DR7; oldD8=DR8; oldD9=DR9; oldD10=DR10;  
}

- (IBAction)undoClear:(id)sender;
{
int x;

if(undoTOG)undoTOG=0; else undoTOG=1;

for(x=0; x < 64; x++){
if(undoTOG)DR1undo2[x] =  DR1undo[x];   else DR1BUF[x]  = DR1undo[x];
if(undoTOG)DR2undo2[x] =  DR2undo[x];   else DR2BUF[x]  = DR2undo[x];
if(undoTOG)DR3undo2[x] =  DR3undo[x];   else DR3BUF[x]  = DR3undo[x];
if(undoTOG)DR4undo2[x] =  DR4undo[x];   else DR4BUF[x]  = DR4undo[x];
if(undoTOG)DR5undo2[x] =  DR5undo[x];   else DR5BUF[x]  = DR5undo[x];
if(undoTOG)DR6undo2[x] =  DR6undo[x];   else DR6BUF[x]  = DR6undo[x];
if(undoTOG)DR7undo2[x] =  DR7undo[x];   else DR7BUF[x]  = DR7undo[x];
if(undoTOG)DR8undo2[x] =  DR8undo[x];   else DR8BUF[x]  = DR8undo[x];
if(undoTOG)DR9undo2[x] =  DR9undo[x];   else DR9BUF[x]  = DR9undo[x];
if(undoTOG)DR10undo2[x] =  DR10undo[x]; else DR10BUF[x] = DR10undo[x];
}
DR1=oldD1; DR2=oldD2; DR3=oldD3; DR4=oldD4; DR5=oldD5; DR6=oldD6; DR7=oldD7; DR8=oldD8; DR9=oldD9; DR10=oldD10;
if(KIT > 1000)[self displayDrums];  else [self displayNotes];

[self setNeedsDisplay:YES];
}



@end
