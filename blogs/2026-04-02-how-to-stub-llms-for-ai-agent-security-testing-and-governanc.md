---
title: "How to Stub LLMs for AI Agent Security Testing and Governance"
url: "https://www.tigera.io/blog/how-to-stub-llms-for-ai-agent-security-testing-and-governance/"
date: "Thu, 02 Apr 2026 14:15:28 +0000"
author: "Alister Baroi"
feed_url: "https://www.tigera.io/feed/"
---
<p style="text-align: center;"><em><strong>Note:</strong> The core architecture for this pattern was introduced by <a href="https://www.linkedin.com/in/isaac-hawley-9481743/">Isaac Hawley</a> from Tigera.</em></p>
<p>If you are building an AI agent that relies on tool calling, complex routing, or the <a href="https://modelcontextprotocol.io/">Model Context Protocol (MCP)</a>, you’re not just building a chatbot anymore. You are building an autonomous system with access to your internal APIs.</p>
<p>With that power comes a massive security and governance headache, and AI agent security testing is where most teams hit a wall. <strong>How do you definitively prove that your agent&#8217;s identity and access management (IAM) actually works?</strong></p>
<p>The scale of the problem is hard to overstate. Microsoft&#8217;s telemetry shows that <a href="https://www.microsoft.com/en-us/security/blog/2026/02/10/80-of-fortune-500-use-active-ai-agents-observability-governance-and-security-shape-the-new-frontier/">80% of Fortune 500 companies now run active AI agents</a>, yet only 47% have implemented specific AI security controls. Most teams are deploying agents faster than they can test them.</p>
<p>If an agent is hijacked via prompt injection, or simply hallucinates a destructive action, does your governance layer stop it? Testing this usually forces engineers into a frustrating trade-off:</p>
<ol>
<li><strong>Use the real API (Gemini, OpenAI):</strong> Real models are heavily RLHF&#8217;d to be safe and polite. It is incredibly difficult (and non-deterministic) to intentionally force a real model to &#8220;go rogue&#8221; and consistently output malicious tool calls so you can test your security boundaries.</li>
<li><strong>Mock the internal tools only:</strong> You test your Python or Go functions in isolation, but you never actually test the &#8220;Agent Loop&#8221;—meaning you aren&#8217;t testing if the harness correctly applies the user&#8217;s OAuth tokens or <a href="https://docs.tigera.io/calico/latest/network-policy/get-started/kubernetes-policy/kubernetes-network-policy">Role-Based Access Control (RBAC)</a> to the LLM&#8217;s requested tool call.</li>
</ol>
<p>Recently, Isaac Hawley introduced a much better pattern: <strong>The Stub Model</strong>—a way to stub your LLM for testing that makes your security assertions completely deterministic.</p>
<p><span class="blockquote">A Stub Model (or mock LLM) is a deterministic, non-AI replacement for a real language model that you inject into your agent harness during testing. It returns hardcoded tool-call requests — including deliberately malicious ones — so you can prove that your security layer correctly intercepts and blocks unauthorized actions without relying on a live model API.</span></p>
<h2>The Core Concept: A &#8220;Malicious&#8221; Router for AI Agent Security Testing</h2>
<p>Instead of hitting a real model API during tests, we inject a <code>StubLLM that</code> implements our system&#8217;s core LLM interface.</p>
<p>The stub doesn&#8217;t use any AI. Instead, it parses incoming prompts for specific testing triggers and returns hardcoded, completely predictable tool calls. Crucially, this forces your agent harness to <strong>actually execute the real underlying tool pipeline</strong>. You aren&#8217;t just faking a final text response; you are making the LLM trigger your application&#8217;s real execution loop.</p>
<p>From a governance perspective, this is a superpower. You can program the stub to request highly privileged actions (like <code>drop_database</code> or<code> read_all_users</code>), and then write strict, lightning-fast assertions to prove that your Agent Harness intercepts the call, checks the executing user&#8217;s identity, and blocks the action.</p>
<p>Here is how you can implement and test this security pattern in both Python and Go.</p>
<h3>Python: Proving RBAC &amp; Tool Governance</h3>
<p>In Python, we use a <code>Protocol</code> to define our LLM dependency, and then build a Stub that intentionally requests unauthorized actions.</p>
<pre class="language-python code-toolbar line-numbers"><code class="language-python">from typing import List, Optional, Protocol
from pydantic import BaseModel
# Define standard tool call response formats
class ToolCall(BaseModel):
   id: str
   name: str
   arguments: dict
class Response(BaseModel):
   content: Optional[str] = None
   tool_calls: Optional[List[ToolCall]] = None
# Define the LLM Interface
class LLMClient(Protocol):
   def generate(self, prompt: str) -&gt; Response:
       ...
# Implement the Stub Model for Security Testing
class StubLLM:
   def generate(self, prompt: str) -&gt; Response:
       # 1. Standard authorized action
       if &quot;MOCK_WEATHER_TOOL&quot; in prompt:
           return Response(
               tool_calls=[ToolCall(id=&quot;call_1&quot;, name=&quot;get_weather&quot;, arguments={&quot;location&quot;: &quot;London&quot;})]
           )
          
       # 2. Malicious / Unauthorized action for Governance testing
       if &quot;MOCK_UNAUTHORIZED_DELETE&quot; in prompt:
            return Response(
               tool_calls=[
                   ToolCall(
                       id=&quot;call_malicious_999&quot;,
                       name=&quot;delete_user_account&quot;,
                       arguments={&quot;user_id&quot;: &quot;admin_01&quot;} # The LLM is trying something dangerous!
                   )
               ]
           )
       return Response(content=&quot;This is a stubbed standard response.&quot;)</code></pre>
<p><strong>The Security Unit Test (<code>pytest</code>):</strong> With the stub in place, we can test that our Agent correctly parses the dangerous tool call, evaluates the user&#8217;s identity, and <strong>blocks</strong> the execution of the real local Python function.</p>
<pre class="language-python code-toolbar line-numbers"><code class="language-python">import pytest
def test_agent_rbac_blocks_unauthorized_tool_execution():
# Arrange: Inject our deterministic stub into the Agent
stubbed_llm = StubLLM()
# Initialize our agent harness with a heavily restricted &quot;guest&quot; identity
agent = Agent(llm_client=stubbed_llm, user_role=&quot;guest_user&quot;)
# Act: Send the trigger that forces our stub to attempt a destructive tool call
response = agent.run(&quot;Please MOCK_UNAUTHORIZED_DELETE&quot;)
# Assert: Verify the Agent&#039;s governance harness intercepted the call,
# checked the &quot;guest_user&quot; identity, and blocked the REAL local tool.
assert response.status == &quot;blocked_by_policy&quot;
assert response.tool_executed is None
assert &quot;Insufficient permissions to execute delete_user_account&quot; in response.error_message</code></pre>
<h3>Go: Validating OAuth &amp; Identity Boundaries</h3>
<p>In Go, this pattern shines for validating complex OAuth scopes or identity propagation in multi-agent networks.</p>
<pre class="language-yaml code-toolbar line-numbers"><code class="language-yaml">package llm
import (
   &quot;encoding/json&quot;
   &quot;strings&quot;
)
type ToolCall struct {
   ID        string `json:&quot;id&quot;`
   Name      string `json:&quot;name&quot;`
   Arguments []byte `json:&quot;arguments&quot;`
}
type Response struct {
   Content   string     `json:&quot;content,omitempty&quot;`
   ToolCalls []ToolCall `json:&quot;tool_calls,omitempty&quot;`
}
type Client interface {
   Generate(prompt string) (*Response, error)
}
type StubLLM struct{}
func NewStubLLM() *StubLLM {
   return &amp;StubLLM{}
}
func (s *StubLLM) Generate(prompt string) (*Response, error) {
   // Simulate an Agent trying to access a secure internal system via MCP
   if strings.Contains(prompt, &quot;MOCK_ACCESS_SECURE_VAULT&quot;) {
       args, _ := json.Marshal(map[string]string{&quot;secret_id&quot;: &quot;prod_db_password&quot;})
      
       return &amp;Response{
           ToolCalls: []ToolCall{
               {
                   ID:        &quot;call_vault_123&quot;,
                   Name:      &quot;read_secure_vault&quot;,
                   Arguments: args,
               },
           },
       }, nil
   }
   return &amp;Response{Content: &quot;Standard response&quot;}, nil
}</code></pre>
<p><strong>The Security Unit Test (<code>testing</code>):</strong> We write a test to guarantee that if the LLM decides to hit the vault, the Agent harness forces the underlying tool to respect the provided OAuth context.</p>
<pre class="language-yaml code-toolbar line-numbers"><code class="language-yaml">package agent_test
import (
&quot;testing&quot;
&quot;errors&quot;
)
func TestAgentEnforcesOAuthScopes(t *testing.T) {
// Arrange: Initialize the agent with the Stub model
stub := llm.NewStubLLM()
// Create an agent context with a standard user OAuth token (No Vault Access)
mockOAuthContext := identity.NewContext(identity.WithScope(&quot;read:public&quot;))
myAgent := agent.New(stub, mockOAuthContext)
// Act: Trigger the LLM to request a highly privileged tool call
result, err := myAgent.Run(&quot;I need you to MOCK_ACCESS_SECURE_VAULT&quot;)
// Assert: Verify the harness evaluated the tool against the OAuth scope and blocked it
if err == nil {
t.Fatalf(&quot;CRITICAL SECURITY FAILURE: Agent executed secure vault tool without proper OAuth scope&quot;)
}
if !errors.Is(err, ErrUnauthorizedToolExecution) {
t.Errorf(&quot;Expected authorization error, got: %v&quot;, err)
}
if result.ExecutedTool == &quot;read_secure_vault&quot; {
t.Errorf(&quot;The real tool was executed despite lack of permissions!&quot;)
}
}</code></pre>
<h2>Why Security &amp; Governance Teams Love This Architecture</h2>
<p>By treating the LLM like any other untrusted external dependency, we achieve total control over our agent&#8217;s testing environment.</p>
<ul>
<li><strong>Auditable Proof of Governance:</strong> You now have concrete CI/CD tests proving that your agent respects OAuth scopes, RBAC, and identity guardrails. You aren&#8217;t just hoping the model behaves; you are proving the harness defends against it when it doesn&#8217;t.</li>
<li><strong>Tests the Real Agent Harness:</strong> Because the LLM returns a perfectly formatted tool call request, your application code actually executes its real security middleware. You validate the entire execution loop, not just a mocked final answer.</li>
<li><strong>Lightning Fast &amp; Free:</strong> You can run thousands of these security edge-case tests in milliseconds without spending a dime on API tokens or exposing secrets in your CI pipeline.</li>
<li><strong>Force Prompt Injection Scenarios:</strong> You can easily stub the LLM to return tool arguments containing SQL injection or XSS payloads to ensure your local tools sanitize inputs provided by the AI.</li>
</ul>
<h2>The Trade-Offs: What the Stub Model DOESN&#8217;T Test</h2>
<p>As powerful as this architecture is for testing your infrastructure, it&#8217;s important to acknowledge that it is not a silver bullet. There are two major things the Stub Model cannot test:</p>
<ol>
<li><strong>It tests the pipes, not the brain:</strong> The stub proves your system can correctly block a malicious tool call, but it does <em>not</em> test whether your system prompt is resilient to <a href="https://www.tigera.io/learn/guides/llm-security/prompt-injection/">prompt injection</a> in the first place. You still need LLM-as-a-judge pipelines and continuous evaluation frameworks to test your model&#8217;s actual reasoning capabilities.</li>
<li><strong>Vendor Schema Drift:</strong> If OpenAI, Anthropic, or Google update the shape of their underlying JSON tool-call schema, your hardcoded stub tests will still pass with flying colors while your production environment crashes. You still need a handful of real, end-to-end (E2E) smoke tests running against the live API on a nightly basis to catch vendor drift.</li>
</ol>
<h2>Beyond the Chatbot: Engineering for Agency</h2>
<p>If you are building complex systems, delegating between autonomous agents, or integrating internal APIs via MCP, you cannot afford to have untested authorization loops.</p>
<p>By treating the LLM like any other untrusted external dependency, we achieve total control over our agent&#8217;s testing environment. We gain <strong>auditable proof of governance</strong>, ensuring we can run thousands of CI/CD tests in milliseconds without exposing secrets or spending a dime on API tokens.</p>
<p>Do yourself a favor: <strong>Stub your LLMs</strong>.</p>
<p><span class="blockquote">Stubbing your LLM proves the guardrails work in test. <strong>TAG</strong> enforces them in production, giving you continuous visibility into every agent action, authorization decision, and policy enforcement event across your entire organization. <a href="https://www.tigera.io/contact-tigera-agent-governance/">Talk to us about TAG</a>.</span></p>
<p>The post <a href="https://www.tigera.io/blog/how-to-stub-llms-for-ai-agent-security-testing-and-governance/">How to Stub LLMs for AI Agent Security Testing and Governance</a> appeared first on <a href="https://www.tigera.io">Tigera – Creator of Calico</a>.</p>
