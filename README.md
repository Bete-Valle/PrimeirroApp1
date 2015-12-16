# PrimeirroApp1
package com.example.bete.primeiroapp;


import android.app.AlertDialog;
import android.app.ListActivity;
import android.app.ProgressDialog;
import android.content.Intent;
import android.net.Uri;
import android.os.AsyncTask;
import android.os.Bundle;
import android.util.Log;
import android.view.View;
import android.widget.ArrayAdapter;
import android.widget.ListView;

import com.google.android.gms.appindexing.Action;
import com.google.android.gms.appindexing.AppIndex;
import com.google.android.gms.common.api.GoogleApiClient;

import org.json.JSONArray;
import org.json.JSONException;
import org.json.JSONObject;

import java.io.BufferedReader;
import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.util.ArrayList;
import java.util.List;

public class ConsumirJsonActivity extends ListActivity {


    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        new DownloadJsonAsyncTask()
                .execute("https://s3-sa-east-1.amazonaws.com/pontotel-docs/data.json");

    }

    @Override
    protected void onListItemClick(ListView l, View v, int position, long id) {
        super.onListItemClick(l, v, position, id);

        Trend trend = (Trend) l.getAdapter().getItem(position);

        Intent it = new Intent(Intent.ACTION_VIEW, Uri.parse(trend.url));
        startActivity(it);

    }

    public class DownloadJsonAsyncTask extends AsyncTask<String, Void, List<Trend>> {

        ProgressDialog dialog;

        @Override
        protected List<Trend> doInBackground(String... params) {
            return null;
        }

        @Override
        protected void onPreExecute() {
            super.onPreExecute();
            dialog = ProgressDialog.show(ConsumirJsonActivity.this, "Aguarde", "Baixando JSON, Por Favor Aguarde...");

        }

        protected List<Trend> doInBackground(String ID, String name, String Pwd) {
            String[] params = new String[0];
            String urlString = params[0];

            HttpClient httpclient = new HttpClient();
            HttpGet httpget = new HttpGet(urlString);

            try {
                HttpResponse response = httpclient.execute(httpget);
                HttpEntity entity = response.getEntity();

                if (entity != null) {
                    InputStream instream = entity.getContent();
                    String json = toString(instream);
                    instream.close();
                    List<Trend> trends = getTrends(json);
                    return trends;
                }
            } catch (Exception e) {
                Log.e("App", "Falha ao acessar Web service", e);
            }
            return null;

        }

        private List<Trend> getTrends(String jsonString) {

            List<Trend> trends = new ArrayList<Trend>();

            try {
                JSONArray trendLists = new JSONArray(jsonString);
                JSONObject trendList = trendLists.getJSONObject(0);
                JSONArray trendsArray = trendList.getJSONArray("trends");
                JSONObject trend;

                for (int i = 0; i < trendsArray.length(); i++) {

                    trend = new JSONObject(trendsArray.getString(i));
                    Log.i("App", "nome=" + trend.getString("name"));
                    Trend objetoTrend = new Trend();
                    objetoTrend.name = trend.getString("name");
                    objetoTrend.url = trend.getString("url");
                    trends.add(objetoTrend);
                }
            } catch (JSONException e) {
                Log.e("App", "Erro no parsing do JSON", e);
            }

            return trends;
        }

        @Override
        protected void onPostExecute(List<Trend> result) {
            super.onPostExecute(result);
            dialog.dismiss();

            if (result.size() > 0) {
                ArrayAdapter<Trend> adapter = new ArrayAdapter<Trend>(
                        ConsumirJsonActivity.this, android.R.layout.simple_list_item_1, result);
                setListAdapter(adapter);
            } else {
                AlertDialog.Builder builder = new AlertDialog.Builder(
                        ConsumirJsonActivity.this).setTitle("Atenção")
                        .setMessage("Não foi possivel acessar essas informções...")
                        .setPositiveButton("OK", null);
                builder.show();
            }
        }

        private String toString(InputStream is) throws IOException {

            byte[] bytes = new byte[1024];
            ByteArrayOutputStream baos = new ByteArrayOutputStream();
            int lidos;

            while ((lidos = is.read(bytes)) > 0) {
                baos.write(bytes, 0, lidos);
            }

            return new String(baos.toByteArray());

        }


    }

    class Trend {

        String name;
        String url;

        @Override
        public String toString() {
            return name;
        }
    }

}
