# Currency-Converter-Main-Activity
package com.Najiba.Wajiha.Mahboob.currencyconverter;

import android.content.Context;
import android.os.AsyncTask;
import android.support.v7.app.ActionBarActivity;
import android.os.Bundle;
import android.text.Editable;
import android.text.TextWatcher;
import android.view.Menu;
import android.view.MenuItem;
import android.view.MotionEvent;
import android.view.View;
import android.view.inputmethod.InputMethodManager;
import android.widget.AdapterView;
import android.widget.EditText;
import android.widget.Spinner;
import android.widget.TextView;
import android.widget.Toast;

import org.apache.http.HttpEntity;
import org.apache.http.HttpResponse;
import org.apache.http.client.ClientProtocolException;
import org.apache.http.client.methods.HttpPost;
import org.apache.http.impl.client.DefaultHttpClient;
import org.apache.http.params.BasicHttpParams;
import org.json.JSONException;
import org.json.JSONObject;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.text.DecimalFormat;
import java.text.NumberFormat;
import java.util.Arrays;


public class MainActivity extends ActionBarActivity {

    EditText convertFromEditText;
    TextView resultTextView;
    TextView updatedTimeTextView;
    Spinner convertFromSpinner;
    Spinner convertToSpinner;
    String currencyFrom = "";
    String currencyTo = "";
    String updateTime = "";
    double currency = 0;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);


        convertFromEditText = (EditText) findViewById(R.id.convert_from_edit_text);
        resultTextView = (TextView) findViewById(R.id.result_text_view);
        updatedTimeTextView = (TextView) findViewById(R.id.updated_time_text_view);
        convertFromSpinner = (Spinner) findViewById(R.id.from_spinner);
        convertToSpinner = (Spinner) findViewById(R.id.to_spinner);

        CurrencyAdapter adapter = new CurrencyAdapter(this, CurrencyInfo.getCurrencyName());
        convertFromSpinner.setAdapter(adapter);
        convertToSpinner.setAdapter(adapter);
        dismissKeyPad(convertFromSpinner);
        dismissKeyPad(convertToSpinner);
        setFromListener(convertFromSpinner);
        setToListener(convertToSpinner);
        convertFromEditText.addTextChangedListener(new TextWatcher() {
            @Override
            public void beforeTextChanged(CharSequence s, int start, int count, int after) {

            }

            @Override
            public void onTextChanged(CharSequence s, int start, int before, int count) {

            }

            @Override
            public void afterTextChanged(Editable s) {
                if (!convertFromEditText.getText().toString().isEmpty()) {
                    new SaveTheFeed().execute();
                } else {
                    resultTextView.setText(getString(R.string.to_text_view));
                }
            }
        });
    }


    public void dismissKeyPad(Spinner spinner) {
        spinner.setOnTouchListener(new View.OnTouchListener() {
            @Override
            public boolean onTouch(View v, MotionEvent event) {
                InputMethodManager imm = (InputMethodManager)
                        getApplicationContext().getSystemService(Context.INPUT_METHOD_SERVICE);
                imm.hideSoftInputFromWindow(convertFromEditText.getWindowToken(), 0);
                return false;
            }
        });
    }

    public void setFromListener(Spinner spinner) {
        spinner.setOnItemSelectedListener(new AdapterView.OnItemSelectedListener() {
            @Override
            public void onItemSelected(AdapterView<?> parent, View view, int position, long id) {
                currencyFrom = CurrencyInfo.getCurrencyName()[position];
            }

            @Override
            public void onNothingSelected(AdapterView<?> parent) {

            }
        });
    }

    public void setToListener(Spinner spinner) {
        spinner.setOnItemSelectedListener(new AdapterView.OnItemSelectedListener() {
            @Override
            public void onItemSelected(AdapterView<?> parent, View view, int position, long id) {
                currencyTo = CurrencyInfo.getCurrencyName()[position];
                new SaveTheFeed().execute();
            }

            @Override
            public void onNothingSelected(AdapterView<?> parent) {

            }
        });
    }

    public void onClickExchange(View view) {
        int fromPos = convertFromSpinner.getSelectedItemPosition();
        int toPos = convertToSpinner.getSelectedItemPosition();
        convertFromSpinner.setSelection(toPos);
        convertToSpinner.setSelection(fromPos);
    }

    class SaveTheFeed extends AsyncTask<Void, Void, Void> {
        String jsonString;
        Double result;

        @Override
        protected Void doInBackground(Void... params) {

            if (!Arrays.asList(CurrencyInfo.getCurrencyName()).contains(currencyFrom)
                    || !Arrays.asList(CurrencyInfo.getCurrencyName()).contains(currencyTo)) {
                Toast.makeText(MainActivity.this, "Error happens :(", Toast.LENGTH_SHORT).show();
            } else {
                DefaultHttpClient httpClient = new DefaultHttpClient(new BasicHttpParams());
                String url = "https://query.yahooapis.com/v1/public/yql" +
                        "?q=select%20*%20from%20yahoo.finance.xchange%20where" +
                        "%20pair%20in%20(%22" + currencyFrom + currencyTo + "%22)&format=json" +
                        "&diagnostics=true&env=store%3A%2F%2Fdatatables.org%2Fa" +
                        "lltableswithkeys&callback=";
                HttpPost httpPost = new HttpPost(url);
                httpPost.setHeader("Content-type", "application/json");
                InputStream inputStream = null;
                try {
                    HttpResponse response = httpClient.execute(httpPost);
                    HttpEntity entity = response.getEntity();
                    inputStream = entity.getContent();
                    BufferedReader reader = new BufferedReader(
                            new InputStreamReader(inputStream, "UTF-8"), 8);
                    StringBuilder sb = new StringBuilder();
                    String line = null;
                    while ((line = reader.readLine()) != null) {
                        sb.append(line);
                    }
                    jsonString = sb.toString();
                    JSONObject jObject = new JSONObject(jsonString).
                            getJSONObject("query").getJSONObject("results").
                            getJSONObject("rate");
                    updateTime = jObject.get("Date").toString() + " " +
                            jObject.get("Time").toString();
                    currency = Double.parseDouble(jObject.get("Rate").toString());
                } catch (ClientProtocolException e) {
                    e.printStackTrace();
                } catch (IOException e) {
                    e.printStackTrace();
                } catch (JSONException e) {
                    e.printStackTrace();
                }
                return null;
            }
            return null;
        }

        @Override
        protected void onPostExecute(Void aVoid) {
                if (convertFromEditText.getText().toString().isEmpty()) {
                    return;
                } else {
                    double fromDecimal =
                        Double.parseDouble(convertFromEditText.getText().toString());
                    result = fromDecimal * currency;
                    NumberFormat format = new DecimalFormat("#0.000");
                    String resultString = format.format(result);
                    String display = fromDecimal + " " + currencyFrom + " = " + resultString
                            + " " + currencyTo;
                    resultTextView.setText(display);
                    updatedTimeTextView.setText(getString(R.string.update_time) + updateTime);
                }
        }
    }

    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        // Inflate the menu; this adds items to the action bar if it is present.
        getMenuInflater().inflate(R.menu.menu_main, menu);
        return true;
    }

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        // Handle action bar item clicks here. The action bar will
        // automatically handle clicks on the Home/Up button, so long
        // as you specify a parent activity in AndroidManifest.xml.
        int id = item.getItemId();

        //noinspection SimplifiableIfStatement
        if (id == R.id.action_settings) {
            return true;
        }

        return super.onOptionsItemSelected(item);
    }

}
