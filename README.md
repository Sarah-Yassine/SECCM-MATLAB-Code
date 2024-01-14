# SECCM-MATLAB-Code
About MATLAB Code to process electrochemical data resulting from SECCM Mapping on Heka systems
Experiment Details droplet_size=80; file1=('your file'); S1=load(file1);

num_landings_y = 15; %Size of grid in y axis num_landings_x = 20; %Size of grid in x axis landing_separation_y=100; %Separation between landing points in y landing_separation_x=100; %Separation between landing points in x Lx=num_landings_xlanding_separation_x; %Length of x axis for map Ly=num_landings_ylanding_separation_y; %Length of y axis for map

%create x,y coordinates %B = repmat(A,n,1) vertical stack of row vector A for n times %x=repmat(0:10:10*(11-1),11,1)

x=repmat(0:landing_separation_x:landing_separation_x*(num_landings_x-1),num_landings_y,1); y=repmat(0:landing_separation_y:landing_separation_y*(num_landings_y-1),num_landings_x,1); x=x'; x=x(:); y=y(:);

1 %create array of zeros Ecorr=zeros(num_landings_xnum_landings_y,1); OCP=zeros(num_landings_xnum_landings_y,1); pit_curr=zeros(num_landings_x*num_landings_y,1);

for k=2:3num_landings_xnum_landings_y %k is the number of experiments: from 1 to 242 for 11*11 scan

format_spec='Trace_1_%d_%d_%d'; %Format data into a string vector

if mod(k,3)==0 %b = mod(a,m) returns the remainder after division of a by m? here k is odd number, means it's LSV
    currents1=S1.(sprintf(format_spec,k,1,3))(:,2); %3 is the last value for any current measurement
    yc1=log10(abs(currents1)); %finidng the log I values
    [icorr1,I1]=min(yc1); %[M,I]=min(A) M is the smallest element in matrix A and I is the row indice of A
    Ecorr(k/3)=S1.(sprintf(format_spec,k,1,4))(I1,2); %4 is the last value for any potential measurement. Ecorr is the potential with smallest current
    pit_curr(k/3)=S1.(sprintf(format_spec,k,1,3))(end,2)*1E9;
end   

if mod(k,3)==2 % when k is odd number it's OCP measurement
    OCP((k+1)/3)=S1.(sprintf(format_spec,k,1,4))(end,2);
end
end

Setting limits %Removing false landing for OCP and PDP curves: a = -0.6; % set the lower Elimit for OCP b = 0.5; % set the upper Elimit for OCP

%Removing false landing for the maps % define OCP limit OCP_limit = [a, b];

% remove outliers OCP(OCP < OCP_limit(1) | OCP > OCP_limit(2)) = NaN; Ecorr(OCP < OCP_limit(1) | OCP >OCP_limit(2) | isnan(OCP)) = NaN; pit_curr(OCP < OCP_limit(1) | OCP >OCP_limit(2) | isnan(OCP)) = NaN;

% Define Approach potential c= -1 fprintf('c = %d V\n', c); % this will print a as -1 V %fprintf('a = %.0f V\n', c); % this will print a as -1.00 V

OCP map f1=figure; scatter(x,y,droplet_size,OCP,'filled'); clim([a b]); % set color axis limits

box on axis([-landing_separation_y Lx -landing_separation_x Ly]) pbaspect([num_landings_x num_landings_y 1])

h=colorbar('eastoutside'); caxis([a b]); set(gca,'FontSize',15,'LineWidth',1.5,'FontWeight','bold') ylabel(h,'E_c_o_r_r(OCP) (V)','FontSize',15); xlabel('X (\mum)','FontSize',16) ylabel('Y (\mum)','FontSize',16)

Ecorr map f1=figure; scatter(x,y,droplet_size,Ecorr,'filled'); caxis([a b]); % set color axis limits

box on axis([-landing_separation_y Lx -landing_separation_x Ly]) pbaspect([num_landings_x num_landings_y 1])

h=colorbar('eastoutside'); set(gca,'FontSize',15,'LineWidth',1.5,'FontWeight','bold') ylabel(h,'E_c_o_r_r(PDP) (V)','FontSize',15); xlabel('X (\mum)','FontSize',16) ylabel('Y (\mum)','FontSize',16)

Histogram of Ecorr (OCP) % Take the first 495 values OCP1 = OCP(1:300);

% Define the bin edges with 0.05 V spacing edges = -1.0:0.05:0.5;

% Compute the histogram counts counts_OCP = histcounts(OCP1, edges);

% Plot the histogram f4=figure; bar(edges(1:end-1), counts_OCP, 'histc'); xlabel('E_{corr} (OCP) /V vs. Ag/AgCl'); ylabel('Count');

set(gca,'FontSize',14,'linewidth',2.0,'Fontweight','bold'); legend(sprintf('E_{app}=%.0fV', c), 'Location','northeast'); xlim([a b]) ylim([0 max(counts_OCP)+5]);

Histogram of Ecorr (PDP) distirbution % Take the first 495 values Ecorr1 = Ecorr(1:200);

% Define the bin edges with 0.05 V spacing edges = -1.0:0.05:0.5;

% Compute the histogram counts counts_Ecorr = histcounts(Ecorr1, edges);

% Plot the histogram f3=figure; bar(edges(1:end-1), counts_Ecorr, 'histc'); xlabel('E_{corr} (PDP) /V vs. Ag/AgCl'); ylabel('Count'); title(' E_{corr} (PDP)')

set(gca,'FontSize',14,'linewidth',2.0,'Fontweight','bold'); legend(sprintf('E_{app}=%.0fV', c), 'Location','northeast'); xlim([a b]) ylim([0 max(counts_Ecorr)+5]);

legend("Position", [0.61,0.79047,0.27767,0.064767])

Combined Histogram % Plot the histograms f12 = figure; bar(edges(1:end-1), [counts_OCP; counts_Ecorr]', 'histc'); xlabel('E_{corr} /V vs. Ag/AgCl'); ylabel('Count'); title('Distribution of E_{corr} (OCP) and E_{corr} (PDP)');

set(gca,'FontSize',14,'linewidth',2.0,'Fontweight','bold'); legend({sprintf('OCP, E_{app}=%.0fV', c), sprintf('PDP, E_{app}=%.0fV', c)}, 'Location','northeast'); xlim([a b])

Compare OCP and Ecorr figure plot1=plot(Ecorr,'.'); set(plot1,'color','k','Markersize',12); hold on plot2=plot(OCP,'.'); set(plot2,'color','r','Markersize',12);

legend('E_c_o_r_r(PDP)','E_c_o_r_r(OCP)','location','northwest'); xlabel('Landing number') ylabel('E_c_o_r_r vs. SCE (V)'); xlim([1 num_landings_x*num_landings_y]) ylim([a b]); set(gca,'FontSize',18,'linewidth',2.0,'Fontweight','bold')

PDP map

f5=figure;

scatter(x,y,droplet_size,pit_curr,'filled');

box on axis([-landing_separation_y Lx -landing_separation_x Ly]) pbaspect([num_landings_x num_landings_y 1]) colormap cool; % set(gca,'layer','top') % shading flat; grid off; h=colorbar('eastoutside'); caxis([1 10]); % set color axis limits

% h.Limits = [30 120];

set(gca,'FontSize',15,'LineWidth',1.5,'FontWeight','bold'); ylabel(h,'I_{max} (nA)','FontSize',15); xlabel('X (\mum)','FontSize',16) ylabel('Y (\mum)','FontSize',16)

OCP Curve %Choose the points to be plotted in ocp and pdp plots f1 = figure; plot_values= [15,35,55,75,105,125,145] for k1= plot_values to_plot = true; % flag to keep track of whether to plot or not k=k1*3-1

    % extract OCP data
   
    ocp_potentials=S1.(sprintf(format_spec,k,1,4))(:,2);
    ocp_times=S1.(sprintf(format_spec,k,1,4))(:,1);

         % find the last potential in the OCP curve
    last_ocp_potential = ocp_potentials(end);

    % check if the last potential is within the desired range
    if last_ocp_potential < a || last_ocp_potential > b
         %if not, do not plot the OCP and PDP curves for this pair of k values
        to_plot = false;
    end

    if to_plot
        % plot OCP and PDP curves if to_plot is true
        legent = sprintf('%d',k1);
        plot(ocp_times, ocp_potentials, 'linewidth', 2, 'DisplayName', legent);
        hold on
        %plot(pdp_times, pdp_potentials, 'linewidth', 2, 'DisplayName', '');
        %hold on

            % mark the corresponding k value as plotted
        ocp_plotted(k1) = true;
    end
end

xlim([0 60]) % Set x-axis range from -1 to 1 ylim([-0.8 0.4]) yticks(-0.7:0.3:0.4) % Set y-axis ticks with 1 value increment xticks(0:30:60) box off

set(gca, 'Box', 'off', 'TickDir', 'out', 'TickLength', [.02 .02], ... 'XMinorTick', 'on', 'YMinorTick', 'off', ... % Turn off YMinorTick 'XColor', 'k', 'YColor', 'k', 'LineWidth', 2, ... 'FontSize', 18, 'linewidth', 2, 'FontWeight', 'bold', ... 'YTickLabel', get(gca, 'YTickLabel')); xlabel('Time (s)', 'FontName', 'Arial', 'FontWeight', 'bold', 'FontSize', 18) ylabel('OCP(V_A_g_/_A_g_C_l)', 'FontName', 'Arial', 'FontWeight', 'bold', 'FontSize', 18)

h = gca; % Get axis to modify h.XAxis.MinorTick = 'on'; % Must turn on minor ticks if they are off h.XAxis.MinorTickValues = -0:15:60; % Minor ticks which don't line up with majors

legend ('show')

PDP Plots f2 = figure; hold on;

for k1 = plot_values to_plot = true; % flag to keep track of whether to plot or not k = k1 * 3 - 1;

% extract PDP data
pdp_currents = S1.(sprintf(format_spec, k + 1, 1, 3))(:, 2);
pdp_potentials = S1.(sprintf(format_spec, k + 1, 1, 4))(:, 2);

% Calculate log(I) and smooth it
y1 = pdp_currents(:);
ys1 = movmean(y1, 10);
ys = log10(abs(ys1));

% Check the condition for not plotting
if ys(1) < -9.5
    to_plot = false;
end

if to_plot
    legend_text = sprintf('%d', k1);
    plot(pdp_potentials, ys, 'linewidth', 2, 'DisplayName', legend_text);
    hold on
 
end
end

xlim([-1.5 1.8]) % Set x-axis range from -1 to 1 ylim([-13 -7.6]) yticks(-13:2:-8) % Set y-axis ticks with 1 value increment xticks(-1.5:1:1.6) % Major ticks

box off

set(gca, 'Box', 'off', 'TickDir', 'out', 'TickLength', [.02 .02], ... 'XMinorTick', 'on', 'YMinorTick', 'off', ... % Turn off YMinorTick 'XColor', 'k', 'YColor', 'k', 'LineWidth', 2, ... 'FontSize',18, 'linewidth', 2, 'FontWeight', 'bold', ... 'YTickLabel', get(gca, 'YTickLabel'));

xlabel('E (V_A_g_/_A_g_C_l)', 'FontName', 'Arial', 'FontWeight', 'bold', 'FontSize', 18,'Color','black') ylabel('log(i)', 'FontName', 'Arial', 'FontWeight', 'bold', 'FontSize', 18,'color','black')

h = gca; % Get axis to modify h.XAxis.MinorTick = 'on'; % Must turn on minor ticks if they are off h.XAxis.MinorTickValues = -1:1:1; % Minor ticks which don't line up with majors legend('show')
