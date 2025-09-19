Used Streamlit to create a Finance web app to track my spending on my credit card. 

Set page title and layout for Streamlit app. 

```ruby
st.set_page_config(page_title='My Finance App', page_icon='💷', layout='wide')
```
<img width="362" height="182" alt="image" src="https://github.com/user-attachments/assets/9cad9a93-c199-4ffe-aa7d-367a2ddf5dfa" />


We then create our main function for the web app, where we create a upload file variable which allows the user to upload any CSV file. Here is where we select our credit card statement. 
```ruby
#load the file
def load_transactions(file): 
    df = pd.read_csv(file)
    df.columns = [col.strip() for col in df.columns] #we want to strip any leading or preceding white spaces from data column headers

    #st.write(df) #This shows the full DF 
    return df


def main():
    st.title('Budgeting App')
    uploaded_file = st.file_uploader('Upload transaction CSV file', type=['csv'])
    if uploaded_file is not None: #if upload file exists 
        df = load_transactions(uploaded_file)
```

<img width="1835" height="299" alt="image" src="https://github.com/user-attachments/assets/b3f2c099-4930-4378-ab5e-37c99ff16ae1" />


Create two df (debits & credits) and split these into two tabs. We'll also create a Category column where we group expenses based on the following rules:

```ruby
def categorise(desc): 
    desc = desc.lower() 

    if 'lidl' in desc or 'tesco' in desc or 'sainsbury' in desc or 'aldi' in desc: 
        return 'Groceries'
    elif 'tfl travel' in desc or 'uber' in desc or 'trainline' in desc: 
        return 'Travel'
    elif 'spotify' in desc or 'energie' in desc or 'vision' in desc: 
        return 'Subscriptions'
    elif 'with energy' in desc or 'virgin media' in desc: 
        return 'Bills'
    elif 'deliveroo' in desc: 
        return 'Take-Away'
    elif 'pele barbers' in desc: 
        return 'Haircut'
    elif 'payment received - thank you' in desc: 
        return 'Direct Debit Payment'
    elif 'amznmktplace' in desc: 
        return 'Online Shopping' 
    else:
```



```ruby
  if df is not None:
            df['Category'] = df['Description'].apply(categorise)

            debits_df = df[df['Amount'] > 0] 
            credits_df = df[df['Amount'] < 0]

            #Add streamlit tabs to our app 
            tab1, tab2 = st.tabs(['Expenses (Debits)', 'Payments (Credits)'])
            with tab1: 
                st.write(debits_df)
```

<img width="1788" height="500" alt="image" src="https://github.com/user-attachments/assets/51e35b69-13dd-42c1-bdd0-062ebee286fa" />


Finally, we'll add an expense summary table and plotly chart visaulisations

```ruby
                #Add visualisations 
                st.subheader('Expense Summary')
                category_totals = debits_df.groupby('Category')['Amount'].sum().reset_index() 
                category_totals = category_totals.sort_values(by='Amount', ascending=False)
                expenses_over_time = debits_df.groupby('Date')['Amount'].sum().reset_index() 
                expenses_over_time = expenses_over_time.sort_values(by='Date', ascending=False)
                
                col1, col2, col3 = st.columns([1, 0.5, 1])
                with col1:
                    #Pie chart visualisation 
                    fig = px.pie(
                        category_totals,
                        values= 'Amount',
                        names= 'Category', 
                        title= 'Expense by Category'
                    )
                    st.plotly_chart(fig, use_container_width=True)

                with col3: 
                    st.dataframe(
                        category_totals, 
                        column_config={'Amount': st.column_config.NumberColumn('Amount', format='%.2f GBP')}, 
                        use_container_width=True, #make table fit width of screen
                        hide_index=True
                    )
                    #Add expense total figure
                    debit_totals = debits_df['Amount'].sum()
                    st.metric('Total Expenses', f'£{debit_totals:,.2f}')

                #Line chart visualisation
                fig = px.area(
                    expenses_over_time, 
                    x='Date', 
                    y='Amount',
                    title='Expenses Through The Month', 
                    markers=True
                )
                st.plotly_chart(fig, use_container_width=True)

            with tab2: 
                st.write(credits_df)
```

<img width="1858" height="778" alt="image" src="https://github.com/user-attachments/assets/93c31a0c-75f1-45a3-930c-8983cfd68247" />


