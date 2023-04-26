
#register
pub async fn add_account(mut req : Request<PgPool>) -> tide::Result<Response> {
    let param : Epass = req.body_json().await?;
    let pool = req.state();
     
     match
     sqlx::query("INSERT INTO public.user (email, password) VALUES ($1,sha256($2));")
     .bind(param.email)
     .bind(param.password.as_bytes())
     .execute(pool).await
     {
        Ok(_x) => {ws_response("OK", "Berhasil insert ke login")},
        Err(e) => {
            println!("error insert : {:?}",e);
            ws_response("Error", "Gagal insert ke login")
        }

     }

}

#login
pub async fn login_account (mut req : Request<PgPool>) -> tide::Result<Response> {
    let param : Epass = req.body_json().await?;
    let pool = req.state();
    let mut resp = Response::new(http::StatusCode::Ok);

    if let Ok(_record) = sqlx::query!(
        "SELECT email FROM public.user WHERE email = $1  and password = sha256($2::text::bytea)",
        param.email,
        param.password,
    ).fetch_one(pool).await{

        let ret = LoginResult{
            status: "Ok".to_string(),
            info: "Login berhasil".to_string(),

        };
        resp.set_status(200);
        resp.set_body(Body::from_json(&ret)?);
    } else {
        let ret = serde_json::json!(
            {
                "status": "Error",
                "info": "Username/password Invalid"
            }
        );
        resp.set_status(http::StatusCode::Ok);
        resp.set_body(Body::from_json(&ret)?);
    }
    Ok(resp)
}
