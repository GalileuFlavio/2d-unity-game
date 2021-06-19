# 2d-unity-game
jogo 2d Unity
using UnityEngine;
using System.Collections;

public class PlayerControl : MonoBehaviour
{
	[HideInInspector]
	public bool facingRight = true;         // Para determinar de que forma o jogador está enfrentando.
    [HideInInspector]
	public bool jump = false;               // Condição para saber se o jogador deve pular.

    public float moveForce = 365f;          // Quantidade de força adicionada para mover o jogador para a esquerda e para a direita.
    public float maxSpeed = 5f;             // O mais rápido que o jogador pode viajar no eixo x.
    public AudioClip[] jumpClips;           // Conjunto de clipes para quando o jogador salta.
    public float jumpForce = 1000f;         // Quantidade de força adicionada quando o jogador salta.
    public AudioClip[] taunts;              // Conjunto de clipes para quando o jogador se toca.
    public float tauntProbability = 50f;    // Possa uma provocação.
    public float tauntDelay = 1f;           // Atrasar para quando a provocação deve acontecer.


    private int tauntIndex;                 // O índice da matriz de provocações indicando a provocação mais recente.
    private Transform groundCheck;          // Uma marcação de posição onde verificar se o jogador está aterrado.
    private bool grounded = false;          // Independentemente de o jogador estar ou não ligado.
    private Animator anim;                  // Referência ao componente do animador do jogador.

    void Awake()
	{
        // Configurando referências.
        groundCheck = transform.Find("groundCheck");
		anim = GetComponent<Animator>();
	}


	void Update()
	{
        //O jogador está aterrado se um linecast na posição groundcheck atinja qualquer coisa na camada terrestre.
        grounded = Physics2D.Linecast(transform.position, groundCheck.position, 1 << LayerMask.NameToLayer("Ground"));

        // Se o botão de salto for pressionado e o jogador estiver aterrado, o jogador deve pular.
        if (Input.GetButtonDown("Jump") && grounded)
			jump = true;
	}


	void FixedUpdate ()
	{
        // Cache a entrada horizontal.
        float h = Input.GetAxis("Horizontal");

        // O parâmetro Speed animator é definido como o valor absoluto da entrada horizontal.
        anim.SetFloat("Speed", Mathf.Abs(h));

        // Se o jogador estiver mudando de direção (h tem um sinal diferente para velocity.x) ou ainda não alcançou o MaxSpeed ...
        if (h * GetComponent<Rigidbody2D>().velocity.x < maxSpeed)
            //  ... adicione uma força ao jogador.
            GetComponent<Rigidbody2D>().AddForce(Vector2.right * h * moveForce);

        // Se a velocidade horizontal do jogador for maior do que o maxSpeed ...
        if (Mathf.Abs(GetComponent<Rigidbody2D>().velocity.x) > maxSpeed)
            // ... ajuste a velocidade do jogador para o maxSpeed no eixo x.
            GetComponent<Rigidbody2D>().velocity = new Vector2(Mathf.Sign(GetComponent<Rigidbody2D>().velocity.x) * maxSpeed, GetComponent<Rigidbody2D>().velocity.y);

        // Se a entrada estiver movendo o jogador para a direita e o jogador estiver virado para a esquerda ...
        if (h > 0 && !facingRight)
			// ... flip the player.
			Flip();
        // Caso contrário, se a entrada estiver movendo o jogador para a esquerda e o jogador estiver virado para a direita ...
        else if (h < 0 && facingRight)
            // ... vire o jogador.
            Flip();

		// If the player should jump...
		if(jump)
		{
			// Set the Jump animator trigger parameter.
			anim.SetTrigger("Jump");

			// Play a random jump audio clip.
			int i = Random.Range(0, jumpClips.Length);
			AudioSource.PlayClipAtPoint(jumpClips[i], transform.position);

			// Add a vertical force to the player.
			GetComponent<Rigidbody2D>().AddForce(new Vector2(0f, jumpForce));

			// Make sure the player can't jump again until the jump conditions from Update are satisfied.
			jump = false;
		}
	}
	
	
	void Flip ()
	{
		// Switch the way the player is labelled as facing.
		facingRight = !facingRight;

		// Multiply the player's x local scale by -1.
		Vector3 theScale = transform.localScale;
		theScale.x *= -1;
		transform.localScale = theScale;
	}


	public IEnumerator Taunt()
	{
		// Check the random chance of taunting.
		float tauntChance = Random.Range(0f, 100f);
		if(tauntChance > tauntProbability)
		{
			// Wait for tauntDelay number of seconds.
			yield return new WaitForSeconds(tauntDelay);

			// If there is no clip currently playing.
			if(!GetComponent<AudioSource>().isPlaying)
			{
				// Choose a random, but different taunt.
				tauntIndex = TauntRandom();

				// Play the new taunt.
				GetComponent<AudioSource>().clip = taunts[tauntIndex];
				GetComponent<AudioSource>().Play();
			}
		}
	}


	int TauntRandom()
	{
		// Choose a random index of the taunts array.
		int i = Random.Range(0, taunts.Length);

		// If it's the same as the previous taunt...
		if(i == tauntIndex)
			// ... try another random taunt.
			return TauntRandom();
		else
			// Otherwise return this index.
			return i;
	}
}
