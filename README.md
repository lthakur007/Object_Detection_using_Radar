SFND_P2_Radar_Target_Generation_and_Detection
Udacity, Sensor Fusion, Project of Radar Target Generation and Detection

Project Layout:
https://github.com/lthakur007/Radar_project/blob/master/ProjectLayout.jfif

--> Radar Specifications used:
Frequency of operation = 77GHz
Max Range = 200m
Range Resolution = 1 m
Max Velocity = 100 m/s

Lc=3*10^8;
rmax=200;
r_res=1;
max_vel=100;



--> User Defined Range and Velocity of target
define the target's initial position=70m and velocity=10m/sec.
Note : Velocity remains contant

Initial_pos=70;
Initial_vel= 10;


--> FMCW Waveform Generation
Design the FMCW waveform by giving the specs of each of its parameters.

Calculated the Bandwidth (B), Chirp Time (Tchirp) and Slope (slope) of the FMCW chirp using the requirements above as shown below.

Operating carrier frequency of Radar
Bsweep=Lc/(2*r_res);
Tchirp = 5.5*2*rmax/Lc;
slope = Bsweep/Tchirp;

--> The number of chirps in one sequence.
--> Its ideal to have 2^value for the ease of running the FFT for Doppler Estimation.
Nd=128; 

--> The number of samples on each chirp.
Nr=1024;

--> Timestamp for running the displacement scenario for every sample on each chirp
t=linspace(0,Nd*Tchirp,Nr*Nd);

--> Creating the vectors for Tx, Rx and Mix based on the total samples input.
Tx=zeros(1,length(t)); 
Rx=zeros(1,length(t));
Mix = zeros(1,length(t));

--> Similar vectors for range_covered and time delay.
r_t=zeros(1,length(t));
td=zeros(1,length(t));

--> Signal generation and Moving Target simulation
for i=1:length(t)         
%     disp(i)
%     disp(length(t))
    % *%TODO* :
    %For each time stamp update the Range of the Target for constant velocity. 
    r_t(i) = Initial_pos + (Initial_vel*t(i));
    td(i) = (2*r_t(i)) / Lc;
    % *%TODO* :
    %For each time sample we need update the transmitted and
    %received signal. 
    Tx(i) = cos(2*pi*(fc*(t(i)) + (0.5*slope*t(i)^2)));
    Rx(i) = cos(2*pi*(fc*(t(i)-td(i)) + (0.5*slope*(t(i)-td(i))^2)));
    
    % *%TODO* :
    %Now by mixing the Transmit and Receive generate the beat signal
    %This is done by element wise matrix multiplication of Transmit and
    %Receiver Signal
    Mix(i) = Tx(i).*Rx(i);
    
end

--> Range Measurement
--> Reshape the vector into Nr*Nd array.
--> Nr and Nd here would also define the size of Range and Doppler FFT respectively.
Mix = reshape(Mix,[Nr,Nd]);

--> run the FFT on the beat signal along the range bins dimension (Nr) and
--> normalize.
--> Take the absolute value of FFT output
--> Output of FFT is double sided signal, but we are interested in only one side of the spectrum.Hence we throw out half of the samples.

Mix_reshp= reshape(Mix,[Nr,Nd]);
sig_fft1 = fft(Mix_reshp,Nr);  
sig_fft1 = sig_fft1./Nr;
sig_fft1 = abs(sig_fft1);
half_sig_fft1 = sig_fft1(1:Nr/2);

--> Plotting the range, plot FFT output

figure ('Name','Range from First FFT')
subplot(2,1,1)
plot(half_sig_fft1);

--> Simulation Result:
