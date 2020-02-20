# correlationFinder

% the correlationFinder searches correlations between different parameters (columns)
% for the investigated cells/datasets (rows) defined in the inputData.xlsx file


%                   R. Herbrechter     20.02.20

%% user parameters (user defined threshold for definition of "relevant" correlations)
significance_niveau = 0.05; %>> p-value threshold
R_threshold = 0.7; %>> correlation coefficient threshold







%% Import the data
[~, ~, inputData] = xlsread('inputData.xlsx','Tabelle1');
inputData(cellfun(@(x) ~isempty(x) && isnumeric(x) && isnan(x),inputData)) = {''};

if exist('inputData')
    fprintf('Data successfully loaded from inputData.xlsx\n\n');
end



%% preparing the correlation task
exloaded = inputData;

exdata1 = exloaded(2:end,3:end); % >> 1 descriptive row as column header and 2 descriptive columns/dataset in the input excel file
exdata2 = cell2mat(exdata1);



%% correlations
fprintf('performing correlations with corrcoef command\n\n');
[R,P] = corrcoef(exdata2);



%% reordering the data into cell array with description.
Rcell1 = num2cell(R);
Pcell1 = num2cell(P);

%headers
Rcell2 = cell(size(Rcell1,1)+1, size(Rcell1,2)+1);
parameter_names = exloaded(1,3:end);
Rcell2(1,2:end) = parameter_names;
Rcell2(2:end,1) = parameter_names';
Pcell2 = Rcell2;

%data
Rcell2(2:end,2:end) = Rcell1;
Pcell2(2:end,2:end) = Pcell1;



%% writing R/Pcell2 to excel file
xlswrite('Rcell2.xlsx', Rcell2);
xlswrite('Pcell2.xlsx', Pcell2);

fprintf('Rcell2.xlsx and Pcell2.xlsx saved\n\n');



%% finding and plotting best correlations

%identifying significant p values
pIndex = zeros(size(P));
pIndex(P<=significance_niveau) = 1;

number_sig = sum(sum(pIndex));
fprintf('%d from %d significant(p<%.3f) values in P correlation matrix found\n\n', number_sig, size(R,1)^2, significance_niveau);


%indentifying relevant correlcoeff.
rIndex = zeros(size(R));
rVektor = sort(reshape(R, [1, (size(R,1)^2)]));

rIndex(abs(R)>=R_threshold) = 1;

numberR = sum(sum(rIndex));
fprintf('%d from %d values above the threshold (%.3f) in R correlation matrix found\n\n', numberR, size(R,1)^2, R_threshold);


%checking those, which hit both criteria
bothAdded = pIndex + rIndex;
bothIndex = zeros(size(R));
bothIndex(bothAdded==2) = 1;

numberBoth = sum(sum(bothIndex));
fprintf('%d from %d values hit both criteria in P and R correlation matrix \n\n', numberBoth, size(R,1)^2);



%% identification of relevant correlations
%listing the groups without double entries and own correlation.

%searching within the relevant part of bothIndex Matrix
hit_cell = cell(2,7);
hit_cell{1,1} = 'comparison_number';
hit_cell{1,2} = 'group1 number';
hit_cell{1,3} = 'group2 number';
hit_cell{1,4} = 'group1';
hit_cell{1,5} = 'group2';
hit_cell{1,6} = 'R';
hit_cell{1,7} = 'P';

ende = 1;
comb_numb = 2;
for x = 2:(size(R,2))
    for y = 1:ende
        if bothIndex(y,x) == 1
            hit_cell{comb_numb,1} = comb_numb;
            hit_cell{comb_numb,2} = y;
            hit_cell{comb_numb,3} = x;
            hit_cell{comb_numb,4} = parameter_names{1,y};
            hit_cell{comb_numb,5} = parameter_names{1,x};
            hit_cell{comb_numb,6} = R(y,x);
            hit_cell{comb_numb,7} = P(y,x);
            comb_numb = comb_numb + 1;
        end
    end 
ende = ende+1;
end



%% writing hit_cell to excel file and saving variables to mat file
xlswrite('hit_cell.xlsx', hit_cell);

save correlationFinder;

fprintf('hit_cell.xlsx saved and run saved in mat file\n\n');


