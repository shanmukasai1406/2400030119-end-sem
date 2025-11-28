import React, { useEffect, useState, useCallback } from "react";
import "./App.css";

export default function App() {
  const [students, setStudents] = useState([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  const [attempt, setAttempt] = useState(0); // increment to retry

  const fetchStudents = useCallback(async (signal) => {
    setLoading(true);
    setError(null);

    try {
      const res = await fetch("https://jsonplaceholder.typicode.com/users", {
        signal,
      });

      if (!res.ok) {
        throw new Error(`Server responded with ${res.status}`);
      }

      const data = await res.json();
      setStudents(data);
    } catch (err) {
      if (err.name === "AbortError") {
        // ignore aborted fetch
        return;
      }
      setError(err.message || "Unknown error");
      setStudents([]);
    } finally {
      setLoading(false);
    }
  }, []);

  useEffect(() => {
    const controller = new AbortController();
    fetchStudents(controller.signal);

    return () => controller.abort();
  }, [fetchStudents, attempt]);

  const handleRetry = () => setAttempt((a) => a + 1);

  return (
    <div className="app">
      <header>
        <h1>Student Data</h1>
      </header>

      <main>
        {loading && (
          <div className="status">
            <div className="spinner" aria-hidden="true" />
            <span>Loading student data...</span>
          </div>
        )}

        {!loading && error && (
          <div className="status error">
            <p>
              <strong>Error:</strong> {error}
            </p>
            <div style={{ marginTop: 8 }}>
              <button className="btn" onClick={handleRetry}>
                Retry
              </button>
            </div>
          </div>
        )}

        {!loading && !error && students.length === 0 && (
          <div className="status">No student data available.</div>
        )}

        {!loading && !error && students.length > 0 && (
          <div className="table-wrap">
            <table className="students-table" role="table">
              <thead>
                <tr>
                  <th>ID</th>
                  <th>Name</th>
                  <th>Username</th>
                  <th>Email</th>
                  <th>City</th>
                  <th>Company</th>
                </tr>
              </thead>

              <tbody>
                {students.map((s) => (
                  <tr key={s.id}>
                    <td>{s.id}</td>
                    <td>{s.name}</td>
                    <td>{s.username}</td>
                    <td>
                      <a href={`mailto:${s.email}`}>{s.email}</a>
                    </td>
                    <td>{s.address?.city ?? "-"}</td>
                    <td>{s.company?.name ?? "-"}</td>
                  </tr>
                ))}
              </tbody>
            </table>
          </div>
        )}
      </main>

      <footer className="footer">
        <small>Data source: JSONPlaceholder (dummy API)</small>
      </footer>
    </div>
  );
}
