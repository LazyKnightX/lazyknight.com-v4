```c#
private Animator m_animator;
private string m_lastDire;

void Start() {
    m_animator = GetComponent<Animator>();
    m_lastDire = null;
}

void Update() {
    m_lastDire = null;
    
    if (Input.GetKey("Down")) { m_lastDire = "down"; }
    if (Input.GetKey("Up")) { m_lastDire = "up"; }
    if (Input.GetKey("Left")) { m_lastDire = "left"; }
    if (Input.GetKey("Right")) { m_lastDire = "right"; }
    
    if (m_lastDire != null)
    {
        m_animator.SetString("Dire", m_lastDire);
    }
}


private string next_dire;
private string state;
void Update()
{
    if (state == "idle")
    {
        return;
    }
    
    if (state == "walk")
    {
        if (next_dire == null)
        {
            next_dire = CalcDirection();
        }
        else
        {
            UpdateMove();
        }
    }
}
void UpdateMove()
{
    if (next_dire == "right")
    {
        transform.Translate(Vector2.right * speed * Time.deltaTime);
        var p = transform.position;
        if (CheckPosition(p) == true)
        {
            state = "idle";
        }
    }
}
 ```
 