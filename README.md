ML_graphs_and_analysis
======================

Graphical representation of subsystem

% function read_gps.m
% This program reads gps and light data in a specified file
% and plots them in various formats.
% programmer: M. Pistacchio, 4/19/14

clear;
filename = 'positionsDemo5.txt';
maxcount = 5000;
% dimension some storage arrays
lat = zeros(1,maxcount);
lon = zeros(1,maxcount);
light = zeros(1,maxcount);

ierr = 0; %
fid = fopen(filename,'r');
fprintf('\n');
fprintf('processing file %s\n',filename);

% reset storage vector counters
count = 1; %count of output variables
word_type = 1; %flag = 1 if lat, 2 if lon, 3 if light

% read each line of text, parse and store into a vector
% FORMAT: Lat.lat lon.lon light.lit 
% 
ncount = 0; %counts lines in file
while 1                         % loop on lines (currently not needed since only one line)
   ncount = ncount+1;           % line counter
   clear params;                % reset the temp input vector
   params = fgets(fid);         %inserts line of text into params
   if (params(1) == -1 )        % break if found end of file
      break;
   end
   lenp = length(params);
   
   % parse each sample (assume first one is latitude)
   ic = 0; %character counter
   bflag = 0; %flag that is set when you've reached the end of the line
   while 1 %looping on the characters in a line
   
      while 1                % find first delimiter
         ic = ic + 1;
         if ic > lenp %check to see if you are at the end
            bflag = 1;
            break;
         end
         ineg = 1; %for normal decodes when the negative sign is on the word
         % find the caret just to the left of a lat, lon or light value
         if params(ic) == '^'
            ic_save1 = ic+1; %this is where the data starts
            break;
         elseif (params(ic) ~= '^' & ic == 1) 
            ic_save1 = ic; %data starts on first (didn't really need since transmitted on one line
            ineg = -1;          % kludge, should make more elegant later (when negative sign not attached)
            break;
         elseif params(ic) == '|' %just completed a set of data points
            count = count + 1;         % increment storage counter
         end
      end
      if bflag == 1
         break;
      end
      while 1                % find second delimiter
         ic = ic+1;
         if ic > lenp
            bflag = 1;
            break;
         end
         if params(ic) == '^'
            ic_save2 = ic-1; %counter needs to be one less than caret location
            break;
         end
         if ic == lenp
            ic_save2 = lenp; %truncate data word if at end of the line without a caret
         end
      end
      if bflag == 1 %break out of outer while
         break;
      end
      
      % store data word into correct vector (based on anticipated type)
      if word_type == 1 %if lat
         lat(count)   = str2num(params(ic_save1:ic_save2));
         word_type = 2;  % for next time, change data type
         ic = ic - 1; %go back one less than caret location
      elseif word_type == 2 %if lon
         lon(count)   = str2num(params(ic_save1:ic_save2));
         lon(count) = lon(count)*ineg;
         word_type = 3;  % for next time
         ic = ic - 1;
      elseif word_type == 3 %if light
         light(count) = str2num(params(ic_save1:ic_save2));
         %Notice: don't push back counter so that you can find the bar
         word_type = 1;  % for next time
      end
      %pause;
      if ic >= lenp         % break if end of line
         break;
      end
      
   end
   
   % if reached the maximum length of storage array
   if count > maxcount
      fprintf('reached maximum length\n');
      break;
   end
   
end
count = count - 1;          % remove last increment

fclose(fid);

% remove any "wild points" from the location data
lat_mean = mean(lat(1:count));
lon_mean = mean(lon(1:count));
for ik=1:count
   if abs(lon(ik)-lon_mean) > 2        % don't allow more than 2 degs of variation (120 nmi!)
      lon(ik) = lon(ik-1);          % just repeat previous "good" value
   end
   if abs(lat(ik)-lat_mean) > 2       % don't allow more than 2 degs of variation (120 nmi!)
      lat(ik) = lat(ik-1);          % just repeat previous "good" value
   end
end

%Plotting
%NOTE: change GEs to google earth coordinates!
%GELon = mean(lon(1:count));
%GELat = mean(lat(1:count));
GELat = 42.730619;
GELon = -73.679814;
figure; %Lon vs. Time Index
hold on;
plot(lon(1:count),'b','linewidth',2);
H = line([1 count],[GELon GELon]); set(H,'color','r'); set(H,'linewidth',3);
hold off;
xlabel('time index');
ylabel('Longitude (degs)');
title('Longitude vs time index');
grid on;
legend('Raw data','Google Earth');

figure; %Lat vs. Time Index
hold on;
plot(lat(1:count),'b','linewidth',2);
H = line([1 count],[GELat GELat]); set(H,'color','r'); set(H,'linewidth',3);
xlabel('time index');
ylabel('Latitude (degs)');
title('Latitude vs time index');
grid on;
legend('Raw data','Google Earth');

figure; %Light vs. Time Index
plot(light(1:count),'b','linewidth',2);
xlabel('time index');
ylabel('Light Amplitude');
title('Light Amplitude vs time index');
grid on;

figure; %Lat vs. Lon - to show the path
plot(lon(1:count),lat(1:count),'b','linewidth',2);
text(lon(1),lat(1),'Start');
text(lon(count),lat(count),'End');
xlabel('Longitude (degs)');
ylabel('Latitude (degs)');
title('Path in Lat/Lon Coords');
grid on;

figure; %Light Intensity vs. Path in Lat/Lon
[lenc widc] = size(jet); 
colormap('jet'); %sets the color map for coding light intensities
Siz(1:count) = 30; %circle size
%Plot xy circles of variable colors
scatter(lon(1:count),lat(1:count),Siz,light(1:count),'filled');
colorbar;
xlabel('Longitude (degs)');
ylabel('Latitude (degs)');
title('Light Level (colored) vs Path in Lat/Lon Coords');
text(lon(1),lat(1),'Start');
text(lon(count),lat(count),'End');
grid on;

% plot locations and light values on a 3-D grid
maxX = 200;
maxY = 200;
maxLon = max(lon(1:count));
minLon = min(lon(1:count));
maxLat = max(lat(1:count));
minLat = min(lat(1:count));
delLon = (maxLon-minLon)/maxX;      % resolution of x grid
delLat = (maxLat-minLat)/maxY;      % resolution of y grid
grid0 = zeros(maxX,maxY);            % set the grid
for ic=1:count
   xdex = round((lon(ic)-minLon)./delLon + delLon/2);
   ydex = round((lat(ic)-minLat)./delLat + delLat/2);
   if xdex > maxX xdex = maxX; end
   if ydex > maxY ydex = maxY; end
   if xdex < 1 xdex = 1; end
   if ydex < 1 ydex = 1; end
   grid0(xdex,ydex) = light(ic);
end
xx(1:maxX) = linspace(minLon,maxLon,maxX);
yy(1:maxY) = linspace(minLat,maxLat,maxY);
figure;
set(gca,'fontsize',14);
mesh(xx,yy,grid0);
xlabel('Longitude');
ylabel('Latitude');
zlabel('Light');
view(-21,38);
title('Light level vs Lat/Lon location');
%axis([minLon*.95 maxLon*1.05 minLat*.95 maxLat*1.05 0 max(light)]);

%what I actually was supposed to do (LOL)
%Empirical Error Distribution Plot (RMS Error)
RMSLat = sqrt((lat(1:count)-GELat).^2);
RMSLon = sqrt((lon(1:count)-GELon).^2);
figure; %Lon RMS Error vs. Time Index
hold on;
plot(RMSLon,'b','linewidth',2);
plot(RMSLat,'r','linewidth',2);
H=line([0 0],[0 0]); set(H,'color','w');
H=line([0 0],[0 0]); set(H,'color','w');
hold off;
xlabel('time index');
ylabel('RMS Error (degs)');
title('RMS Error vs time index');
grid on;
legend('Longitude','Latitude', ...
    ['GELon = ',num2str(GELon),' degs'],['GELat = ',num2str(GELat),' degs']);

ERRLat = lat(1:count)-GELat;
ERRLon = lon(1:count)-GELon;
figure; %Lon RMS Error vs. Time Index
hold on;
plot(ERRLon,'b','linewidth',2);
plot(ERRLat,'r','linewidth',2);
H=line([0 0],[0 0]); set(H,'color','w');
H=line([0 0],[0 0]); set(H,'color','w');
hold off;
xlabel('time index');
ylabel('Error (degs)');
title('Error vs time index');
grid on;
legend('Longitude','Latitude', ...
    ['GELon = ',num2str(GELon),' degs'],['GELat = ',num2str(GELat),' degs']);

%Histogram of Lat/Lon Error
histERRdata = sqrt(((lat(1:count)-GELat).^2)+((lon(1:count)-GELon).^2));
figure;
M=50; %number of bins
hist(histERRdata,M);
xlabel('Error (degs)');
ylabel('Frequency at which error is observed');
title('Histogram of Latitude and Longitude Error');
grid on;

for I=1:count
if histERRdata == max(histERRdata)
fprintf('\d,\d\n'),lat(I),lon(I);
end
end
