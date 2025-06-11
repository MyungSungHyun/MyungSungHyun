using System.Collections;
using UnityEngine;

public class Moving : MonoBehaviour
{
    public float moveSpeed = 1f;
    public float jumpHeight = 5f;
    public int maxJumpCount = 2;
    public float dashCooltime = 1.5f;

    public Transform groundTransform;
    public LayerMask groundLayer;
    public float dashTime = 0.2f;
    public float dashForce = 12f;

    public bool canMove = true;

    public GameObject dashAftereffect;

    Vector2 lastMoveDir;
    float nextDashTime;

    Animator animator;
    SpriteRenderer sr;
    int jumpCountLeft = 0;
    bool isGrounded;
    bool wasGrounded;
    bool isDashing;

    Rigidbody2D rigid;
    void Start()
    {
        rigid = GetComponent<Rigidbody2D>();
        sr = GetComponentInChildren<SpriteRenderer>();
        animator = GetComponentInChildren<Animator>();
    }

    private void OnDrawGizmosSelected()
    {
        Gizmos.color = Color.yellow;
        if (groundTransform)
            Gizmos.DrawWireSphere(groundTransform.position, 0.1f);
    }

    void Update()
    {
        isGrounded = Physics2D.OverlapCircle(groundTransform.position, 0.1f, groundLayer);
        
        if (!wasGrounded && isGrounded)
            jumpCountLeft = maxJumpCount;
        animator.SetBool("isGrounded", isGrounded);
        
        if (!canMove)
            return;

        if (!isDashing) {
            Move();
            Vector2 moveDir = new Vector2(Input.GetAxisRaw("Horizontal"), Input.GetAxisRaw("Vertical"));
            if (moveDir != Vector2.zero)
                lastMoveDir = moveDir;

            if (Input.GetKeyDown(KeyCode.LeftShift) && Time.time > nextDashTime)
            {
                Dash();
            }
            if (Input.GetKeyDown(KeyCode.Space) && jumpCountLeft > 0)
            {
                Jump();
            }
        }
        wasGrounded = isGrounded;
    }
    void Dash()
    {
        nextDashTime = Time.time + dashCooltime;
        StartCoroutine(DashMotion());
    }
    IEnumerator DashMotion()
    {
        isDashing = true;
        float delta = 0;
        float effectTime = 0;
        while (delta < dashTime)
        {
            rigid.linearVelocity = lastMoveDir * dashForce;
            yield return null;
            delta += Time.deltaTime;
            if (delta > effectTime)
            {
                effectTime += dashTime * 0.1f;
                GameObject go = Instantiate(dashAftereffect, sr.transform.position, Quaternion.identity);
                go.GetComponent<SpriteRenderer>().sprite = sr.sprite;
            }
        }
        rigid.linearVelocity = lastMoveDir * moveSpeed * 0.5f;
        isDashing = false;
    }
    void Move()
    {
        float xInput = Input.GetAxisRaw("Horizontal");
        if (xInput < 0)
            sr.flipX = true;
        else if (xInput > 0)
            sr.flipX = false;

        rigid.linearVelocity = new Vector2(xInput * moveSpeed, rigid.linearVelocity.y);
        animator.SetBool("isMoving", xInput != 0);
    }
    void Jump()
    {
        jumpCountLeft--;
        rigid.linearVelocity = new Vector2(rigid.linearVelocity.x, Mathf.Sqrt(2 * 9.81f * jumpHeight * rigid.gravityScale));
    }
}
